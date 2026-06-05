# bench_dspy_intro.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/dspy/bench_dspy_intro.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on dspy intro. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and loads, filters, or serializes benchmark datasets. / 该 Python 模块聚焦于 dspy intro 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、加载、筛选或序列化基准测试数据集。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Imports and setup / 导入与初始化
```python
"""
Adapted from
https://github.com/stanfordnlp/dspy/blob/34d8420383ec752037aa271825c1d3bf391e1277/intro.ipynb#L9
"""

import argparse

import dspy
from dspy.datasets import HotPotQA
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集。

### Lines 12-12: Class `BasicQA` declaration / 类 `BasicQA` 声明
```python
class BasicQA(dspy.Signature):
```
**EN:** This block introduces class `BasicQA`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `BasicQA`，用于通过统一接口组织相关行为。

### Lines 13-16: Class-level state / 类级状态
```python
    """Answer questions with short factoid answers."""

    question = dspy.InputField()
    answer = dspy.OutputField(desc="often between 1 and 5 words")
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it computes evaluation scores and aggregate statistics.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会计算评测分数与聚合统计结果。

### Lines 19-19: Class `GenerateAnswer` declaration / 类 `GenerateAnswer` 声明
```python
class GenerateAnswer(dspy.Signature):
```
**EN:** This block introduces class `GenerateAnswer`, which computes evaluation scores and aggregate statistics.
**CN:** 该代码块引入类 `GenerateAnswer`，用于计算评测分数与聚合统计结果。

### Lines 20-24: Class-level state / 类级状态
```python
    """Answer questions with short factoid answers."""

    context = dspy.InputField(desc="may contain relevant facts")
    question = dspy.InputField()
    answer = dspy.OutputField(desc="often between 1 and 5 words")
```
**EN:** This block stores class-level metadata, constants, or a docstring that explains how the surrounding class is organized. In practice it computes evaluation scores and aggregate statistics.
**CN:** 该代码块保存类级元数据、常量，或通过文档字符串说明外围类的组织方式。 从实现上看，它会计算评测分数与聚合统计结果。

### Lines 27-27: Class `RAG` declaration / 类 `RAG` 声明
```python
class RAG(dspy.Module):
```
**EN:** This block introduces class `RAG`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `RAG`，用于通过统一接口组织相关行为。

### Lines 28-32: Method `RAG.__init__` / 方法 `RAG.__init__`
```python
    def __init__(self, num_passages=3):
        super().__init__()

        self.retrieve = dspy.Retrieve(k=num_passages)
        self.generate_answer = dspy.ChainOfThought(GenerateAnswer)
```
**EN:** `RAG.__init__` is a method that computes evaluation scores and aggregate statistics. Notable calls include `super().__init__`, `dspy.Retrieve`, `dspy.ChainOfThought`.
**CN:** `RAG.__init__` 是一个方法，用于计算评测分数与聚合统计结果。其中较关键的调用包括 `super().__init__`, `dspy.Retrieve`, `dspy.ChainOfThought`。

### Lines 34-37: Method `RAG.forward` / 方法 `RAG.forward`
```python
    def forward(self, question):
        context = self.retrieve(question).passages
        prediction = self.generate_answer(context=context, question=question)
        return dspy.Prediction(context=context, answer=prediction.answer)
```
**EN:** `RAG.forward` is a method that computes evaluation scores and aggregate statistics. It returns `dspy.Prediction(context=context, answer=prediction.answer)` to the caller. Notable calls include `self.generate_answer`, `dspy.Prediction`, `self.retrieve`.
**CN:** `RAG.forward` 是一个方法，用于计算评测分数与聚合统计结果。它会向调用方返回 `dspy.Prediction(context=context, answer=prediction.answer)`。其中较关键的调用包括 `self.generate_answer`, `dspy.Prediction`, `self.retrieve`。

### Lines 40-170: Function `main` / 函数 `main`
```python
def main(args):
    # lm = dspy.OpenAI(model='gpt-3.5-turbo')
    if args.backend == "tgi":
        lm = dspy.HFClientTGI(
            model="meta-llama/Llama-2-7b-chat-hf",
            port=args.port,
            url="http://localhost",
        )
    elif args.backend == "sglang":
        lm = dspy.HFClientSGLang(
            model="meta-llama/Llama-2-7b-chat-hf",
            port=args.port,
            url="http://localhost",
        )
    elif args.backend == "vllm":
        lm = dspy.HFClientVLLM(
            model="meta-llama/Llama-2-7b-chat-hf",
            port=args.port,
            url="http://localhost",
        )
    else:
        raise ValueError(f"Invalid backend: {args.backend}")

    colbertv2_wiki17_abstracts = dspy.ColBERTv2(
        url="http://20.102.90.50:2017/wiki17_abstracts"
    )
    dspy.settings.configure(lm=lm, rm=colbertv2_wiki17_abstracts)

    # Load the dataset.
    dataset = HotPotQA(
        train_seed=1, train_size=20, eval_seed=2023, dev_size=args.dev_size, test_size=0
    )

    # Tell DSPy that the 'question' field is the input. Any other fields are labels and/or metadata.
    trainset = [x.with_inputs("question") for x in dataset.train]
    devset = [x.with_inputs("question") for x in dataset.dev]

    print(len(trainset), len(devset))

    train_example = trainset[0]
    print(f"Question: {train_example.question}")
    print(f"Answer: {train_example.answer}")

    dev_example = devset[18]
    print(f"Question: {dev_example.question}")
    print(f"Answer: {dev_example.answer}")
    print(f"Relevant Wikipedia Titles: {dev_example.gold_titles}")

    print(
        f"For this dataset, training examples have input keys {train_example.inputs().keys()} and label keys {train_example.labels().keys()}"
    )
    print(
        f"For this dataset, dev examples have input keys {dev_example.inputs().keys()} and label keys {dev_example.labels().keys()}"
    )

    # Define the predictor.
    generate_answer = dspy.Predict(BasicQA)

    # Call the predictor on a particular input.
    pred = generate_answer(question=dev_example.question)

    # Print the input and the prediction.
    print(f"Question: {dev_example.question}")
    print(f"Predicted Answer: {pred.answer}")

    lm.inspect_history(n=1)

    # Define the predictor. Notice we're just changing the class. The signature BasicQA is unchanged.
    generate_answer_with_chain_of_thought = dspy.ChainOfThought(BasicQA)

    # Call the predictor on the same input.
    pred = generate_answer_with_chain_of_thought(question=dev_example.question)

    # Print the input, the chain of thought, and the prediction.
    print(f"Question: {dev_example.question}")
    print(f"Thought: {pred.rationale.split('.', 1)[1].strip()}")
    print(f"Predicted Answer: {pred.answer}")

    retrieve = dspy.Retrieve(k=3)
    topK_passages = retrieve(dev_example.question).passages

    print(
        f"Top {retrieve.k} passages for question: {dev_example.question} \n",
        "-" * 30,
        "\n",
    )

    for idx, passage in enumerate(topK_passages):
        print(f"{idx+1}]", passage, "\n")

    retrieve("When was the first FIFA World Cup held?").passages[0]

    from dspy.teleprompt import BootstrapFewShot

    # Validation logic: check that the predicted answer is correct.
    # Also check that the retrieved context does actually contain that answer.
    def validate_context_and_answer(example, pred, trace=None):
        answer_EM = dspy.evaluate.answer_exact_match(example, pred)
        answer_PM = dspy.evaluate.answer_passage_match(example, pred)
        return answer_EM and answer_PM

    # Set up a basic teleprompter, which will compile our RAG program.
    teleprompter = BootstrapFewShot(metric=validate_context_and_answer)

    # Compile!
    compiled_rag = teleprompter.compile(RAG(), trainset=trainset)

    # Ask any question you like to this simple RAG program.
    my_question = "What castle did David Gregory inherit?"

    # Get the prediction. This contains `pred.context` and `pred.answer`.
    pred = compiled_rag(my_question)

    # Print the contexts and the answer.
    print(f"Question: {my_question}")
    print(f"Predicted Answer: {pred.answer}")
    print(f"Retrieved Contexts (truncated): {[c[:200] + '...' for c in pred.context]}")

    from dspy.evaluate.evaluate import Evaluate

    # Set up the `evaluate_on_hotpotqa` function. We'll use this many times below.
    evaluate_on_hotpotqa = Evaluate(
        devset=devset,
        num_threads=args.num_threads,
        display_progress=True,
        display_table=5,
    )

    # Evaluate the `compiled_rag` program with the `answer_exact_match` metric.
    metric = dspy.evaluate.answer_exact_match
    evaluate_on_hotpotqa(compiled_rag, metric=metric)
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, loads, filters, or serializes benchmark datasets, and computes evaluation scores and aggregate statistics. It returns `answer_EM and answer_PM` to the caller. Notable calls include `dspy.ColBERTv2`, `dspy.settings.configure`, `HotPotQA`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。它会向调用方返回 `answer_EM and answer_PM`。其中较关键的调用包括 `dspy.ColBERTv2`, `dspy.settings.configure`, `HotPotQA`。

### Lines 173-192: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--port", type=int)
    parser.add_argument("--num-threads", type=int, default=32)
    parser.add_argument("--dev-size", type=int, default=150)
    parser.add_argument(
        "--backend", type=str, choices=["sglang", "tgi", "vllm"], default="sglang"
    )
    args = parser.parse_args()

    if args.port is None:
        default_port = {
            "vllm": 21000,
            "lightllm": 22000,
            "tgi": 24000,
            "sglang": 30000,
        }
        args.port = default_port.get(args.backend, None)

    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`
- **Third-party / 第三方依赖**: `dspy`, `dspy.datasets`, `dspy.teleprompt`, `dspy.evaluate.evaluate`

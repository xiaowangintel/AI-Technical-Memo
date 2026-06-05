# custom_req.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/logits_processor/custom_req.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This example demonstrates wrapping a request-level logits processor to be compatible with vLLM's batch-level logits processing For demo purposes, a dummy logits processor is employed which, if `target_token` is passed / 演示自定义 logits 处理钩子。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""This example demonstrates wrapping a request-level logits processor to be
compatible with vLLM's batch-level logits processing

For demo purposes, a dummy logits processor is employed which, if
`target_token` is passed as a keyword argument to `SamplingParams.extra_args`,
will mask out all tokens except `target_token`. This logits processor can be
applied to a vector of logits associated with a single decode step for a single
request. The logits processor cannot be applied to a request which does not
pass in a `target_token` custom argument.

The request-level dummy logits processor is wrapped to create a batch-level
logits processor, which can apply the logits processor to output logits from
# ... key logic omitted for brevity ...
Prompt:    'The capital of France is'
Output:    ' also also also also also also also also also also also also also
             also also also'
------------------------------------------------------------
Prompt:    'The future of AI is'
Output:    ' in the hands of the people.\n\nThe future of AI is in the'
------------------------------------------------------------
"""
```
**EN:** This example demonstrates wrapping a request-level logits processor to be compatible with vLLM's batch-level logits processing For demo purposes, a dummy logits processor is employed which, if `target_token` is passed...
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from typing import Any

import torch

from vllm import LLM, SamplingParams
from vllm.logger import init_logger
from vllm.v1.sample.logits_processor import (
    AdapterLogitsProcessor,
    RequestLogitsProcessor,
)
```
**EN:** This block loads helper libraries such as typing and torch and pulls in vLLM APIs like vllm, vllm.logger, and vllm.v1.sample.logits_processor.
**CN:** 这一部分加载 typing 和 torch 等辅助库，并引入 vllm、vllm.logger，以及 vllm.v1.sample.logits_processor 等 vLLM API。

### Top-level setup
```python
logger = init_logger(__name__)
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as logger. It also performs early helper calls such as init_logger.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 logger 等变量。它还会提前执行 init_logger 等辅助调用。

### Class: DummyPerReqLogitsProcessor
```python
class DummyPerReqLogitsProcessor:
    """The request-level logits processor masks out all logits except the
    token id identified by `target_token`"""

    def __init__(self, target_token: int) -> None:
        """Specify `target_token`"""
        self.target_token = target_token

    def __call__(
        self,
        output_ids: list[int],
        logits: torch.Tensor,
    ) -> torch.Tensor:
        val_to_keep = logits[self.target_token].item()
        logits[:] = float("-inf")
        logits[self.target_token] = val_to_keep
        return logits
```
**EN:** The request-level logits processor masks out all logits except the token id identified by `target_token`. Notable methods include __init__ and __call__.
**CN:** 该类封装了示例中使用的 DummyPerReqLogitsProcessor 抽象。较重要的方法包括 __init__ 和 __call__。

### Class: WrappedPerReqLogitsProcessor
```python
class WrappedPerReqLogitsProcessor(AdapterLogitsProcessor):
    """Example of wrapping a fake request-level logit processor to create a
    batch-level logits processor"""

    @classmethod
    def validate_params(cls, params: SamplingParams):
        target_token: Any | None = params.extra_args and params.extra_args.get(
            "target_token"
        )
        if target_token is not None and not isinstance(target_token, int):
            raise ValueError(f"target_token value {target_token} is not int")

    def is_argmax_invariant(self) -> bool:
        return False

    def new_req_logits_processor(
        self,
        params: SamplingParams,
    ) -> RequestLogitsProcessor | None:
        """This method returns a new request-level logits processor, customized
        to the `target_token` value associated with a particular request.

        Returns None if the logits processor should not be applied to the
        particular request. To use the logits processor the request must have
        a "target_token" custom argument with an integer value.

        Args:
          params: per-request sampling params

        Returns:
          `Callable` request logits processor, or None
        """
        target_token: Any | None = params.extra_args and params.extra_args.get(
            "target_token"
        )
        if target_token is None:
            return None
        return DummyPerReqLogitsProcessor(target_token)
```
**EN:** Example of wrapping a fake request-level logit processor to create a batch-level logits processor. It extends AdapterLogitsProcessor. Notable methods include validate_params, is_argmax_invariant, and new_req_logits_processor.
**CN:** 该类封装了示例中使用的 WrappedPerReqLogitsProcessor 抽象。它继承自 AdapterLogitsProcessor。较重要的方法包括 validate_params、is_argmax_invariant，以及 new_req_logits_processor。

### Top-level setup
```python
prompts = [
    "Hello, my name is",
    "The president of the United States is",
    "The capital of France is",
    "The future of AI is",
]
# Create a mixture of requests which do and don't utilize the dummy logitproc
sampling_params_list = [
    SamplingParams(temperature=0.0, extra_args={"target_token": 128}),
    SamplingParams(temperature=0.0),
    SamplingParams(temperature=0.0, extra_args={"target_token": 67}),
    SamplingParams(temperature=0.0),
]
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as prompts and sampling_params_list. It also performs early helper calls such as SamplingParams.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 prompts 和 sampling_params_list 等变量。它还会提前执行 SamplingParams 等辅助调用。

### Function: main
```python
def main():
    # Create an LLM.
    llm = LLM(
        model="facebook/opt-125m",
        logits_processors=[WrappedPerReqLogitsProcessor],
    )
    # Generate texts from the prompts.
    # The output is a list of RequestOutput objects
    # that contain the prompt, generated text, and other information.
    outputs = llm.generate(prompts, sampling_params_list)
    # Print the outputs.
    print("\nGenerated Outputs:\n" + "-" * 60)
    for output in outputs:
        prompt = output.prompt
        generated_text = output.outputs[0].text
        print(f"Prompt:    {prompt!r}")
        print(f"Output:    {generated_text!r}")
        print("-" * 60)
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include print, LLM, and llm.generate.
**CN:** 该函数编排端到端工作流。关键操作包括 print、LLM，以及 llm.generate。

### Entry point
```python
if __name__ == "__main__":
    main()
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **Disaggregated execution / 解耦执行**: Prefill, decode, or cache components are split across services or processes. / 预填充、解码或缓存组件被拆分到不同服务或进程中。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.logger`, `vllm.v1.sample.logits_processor` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `typing`, `torch` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `DummyPerReqLogitsProcessor`, `WrappedPerReqLogitsProcessor`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `init_logger`, `item`, `float`, `params.extra_args.get`, `DummyPerReqLogitsProcessor`, `ValueError`, `isinstance`, `SamplingParams` reveal the main execution path / 这些调用体现了主要执行链路。

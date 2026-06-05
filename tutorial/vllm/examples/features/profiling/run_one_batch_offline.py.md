# run_one_batch_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/profiling/run_one_batch_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Demonstrates the run one batch offline workflow in the vllm examples tree. / 演示 vLLM 示例目录中与 run one batch offline 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Imports
```python
from __future__ import annotations

from vllm import LLM, EngineArgs
from vllm.config import ProfilerConfig
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block loads helper libraries such as __future__ and pulls in vLLM APIs like vllm, vllm.config, and vllm.utils.argparse_utils.
**CN:** 这一部分加载 __future__ 等辅助库，并引入 vllm、vllm.config，以及 vllm.utils.argparse_utils 等 vLLM API。

### Top-level setup
```python
DEFAULT_MAX_TOKENS = 16
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as DEFAULT_MAX_TOKENS.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 DEFAULT_MAX_TOKENS 等变量。

### Function: create_parser
```python
def create_parser() -> FlexibleArgumentParser:
    parser = FlexibleArgumentParser()
    EngineArgs.add_cli_args(parser)
    parser.set_defaults(model="meta-llama/Llama-3.2-1B-Instruct")

    batch_group = parser.add_argument_group("Batch parameters")
    batch_group.add_argument("--batch-size", type=int, default=1)
    batch_group.add_argument("--prompt-size", type=int, default=128)
    batch_group.add_argument("--prompt-prefix", type=str, default="Hello, my name is")

    profile_group = parser.add_argument_group("Profiling parameters")
    profile_group.add_argument(
        "--profile",
        choices=["none", "prefill", "decode", "both"],
        default="none",
    )
    profile_group.add_argument(
        "--profile-dir",
        type=str,
        default="",
        help="Required when --profile is not 'none'.",
    )

    return parser
```
**EN:** This function builds and validates command-line arguments. Key operations include batch_group.add_argument, parser.add_argument_group, profile_group.add_argument, FlexibleArgumentParser, and EngineArgs.add_cli_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 batch_group.add_argument、parser.add_argument_group、profile_group.add_argument、FlexibleArgumentParser，以及 EngineArgs.add_cli_args。其返回值会继续传给示例管线的下一阶段。

### Function: _build_prompt
```python
def _build_prompt(prefix: str, prompt_size: int) -> str:
    if prompt_size <= 0:
        return ""
    if not prefix:
        prefix = " "
    if len(prefix) >= prompt_size:
        return prefix[:prompt_size]
    repeat_count = (prompt_size + len(prefix) - 1) // len(prefix)
    return (prefix * repeat_count)[:prompt_size]
```
**EN:** This function constructs a core runtime component. It works with parameters such as prefix and prompt_size. Key operations include len. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。它会处理 prefix 和 prompt_size 等参数。关键操作包括 len。其返回值会继续传给示例管线的下一阶段。

### Function: _build_profiler_config
```python
def _build_profiler_config(
    profile: str, profile_dir: str, max_tokens: int
) -> ProfilerConfig | None:
    if profile == "none":
        return None
    if not profile_dir:
        raise ValueError("--profile-dir must be set when profiling is enabled.")
    if profile == "prefill":
        delay_iterations = 0
        max_iterations = 1
    elif profile == "decode":
        delay_iterations = 1
        max_iterations = max(1, max_tokens)
    else:
        delay_iterations = 0
        max_iterations = 0

    return ProfilerConfig(
        profiler="torch",
        torch_profiler_dir=profile_dir,
        delay_iterations=delay_iterations,
        max_iterations=max_iterations,
    )
```
**EN:** This function constructs a core runtime component. It works with parameters such as profile, profile_dir, and max_tokens. Key operations include ProfilerConfig, ValueError, and max. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。它会处理 profile、profile_dir，以及 max_tokens 等参数。关键操作包括 ProfilerConfig、ValueError，以及 max。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args: dict) -> None:
    max_tokens = DEFAULT_MAX_TOKENS
    batch_size = args.pop("batch_size")
    prompt_size = args.pop("prompt_size")
    prompt_prefix = args.pop("prompt_prefix")
    profile = args.pop("profile")
    profile_dir = args.pop("profile_dir")

    profiler_config = _build_profiler_config(profile, profile_dir, max_tokens)
    if profiler_config is not None:
        args["profiler_config"] = profiler_config

    llm = LLM(**args)

    sampling_params = llm.get_default_sampling_params()
    sampling_params.max_tokens = max_tokens
    sampling_params.min_tokens = max_tokens
    sampling_params.ignore_eos = True

    prompt = _build_prompt(prompt_prefix, prompt_size)
    prompts = [prompt] * batch_size

    if profile != "none":
        llm.start_profile()
    outputs = llm.generate(prompts, sampling_params)
    if profile != "none":
        llm.stop_profile()

    print("-" * 50)
    for output in outputs:
        generated_text = output.outputs[0].text
        print(f"Prompt: {output.prompt!r}\nGenerated text: {generated_text!r}")
        print("-" * 50)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include args.pop, print, _build_profiler_config, LLM, and llm.get_default_sampling_params.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 args.pop、print、_build_profiler_config、LLM，以及 llm.get_default_sampling_params。

### Entry point
```python
if __name__ == "__main__":
    parser = create_parser()
    main(vars(parser.parse_args()))
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to create_parser, main, vars, and parser.parse_args.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 create_parser、main、vars，以及 parser.parse_args。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **Disaggregated execution / 解耦执行**: Prefill, decode, or cache components are split across services or processes. / 预填充、解码或缓存组件被拆分到不同服务或进程中。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.config`, `vllm.utils.argparse_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `__future__` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `create_parser`, `_build_prompt`, `_build_profiler_config`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `batch_group.add_argument`, `parser.add_argument_group`, `profile_group.add_argument`, `FlexibleArgumentParser`, `EngineArgs.add_cli_args`, `parser.set_defaults`, `len`, `ProfilerConfig` reveal the main execution path / 这些调用体现了主要执行链路。

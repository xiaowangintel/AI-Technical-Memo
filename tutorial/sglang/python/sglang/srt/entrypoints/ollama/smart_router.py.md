# smart_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/ollama/smart_router.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements smart router logic for runtime entrypoints and service adapters. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 smart 路由器 相关的逻辑，并服务于 运行时入口与服务适配层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16: Provide supporting module logic / 提供辅助模块逻辑
```python
"""
Smart Router: Automatically routes requests between local Ollama and remote SGLang.

Uses an LLM judge to classify tasks as simple or complex, then routes accordingly:
- Simple tasks → Local Ollama (fast response)
- Complex tasks → Remote SGLang (powerful model)

Usage:
    from sglang.srt.entrypoints.ollama.smart_router import SmartRouter

    router = SmartRouter(
        local_host="http://localhost:11434",
        remote_host="http://sglang-server:30001",
    )
    response = router.chat("Hello!")
"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 18-18: Import runtime dependencies / 导入运行时依赖
```python
from typing import Optional
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 20-20: Import runtime dependencies / 导入运行时依赖
```python
import ollama
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 23-37: Provide supporting module logic / 提供辅助模块逻辑
```python
class SmartRouter:
    """Routes requests between local Ollama and remote SGLang using LLM-based classification."""

    # Classification prompt for LLM judge
    CLASSIFICATION_PROMPT = """You are a task classifier. Classify the following user request into one of two categories.

Categories:
- SIMPLE: Quick responses, greetings, factual questions, definitions, translations, basic Q&A
- COMPLEX: Tasks requiring deep reasoning, multi-step analysis, long explanations, creative writing, detailed research

Reply with ONLY one word: either SIMPLE or COMPLEX.

User request: "{prompt}"

Category:"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 39-67: Initialize SmartRouter / 初始化 SmartRouter
```python
    def __init__(
        self,
        local_host: str = "http://localhost:11434",
        remote_host: str = "http://localhost:30001",
        local_model: str = "llama3.2",
        remote_model: str = "Qwen/Qwen2.5-1.5B-Instruct",
        judge_model: Optional[str] = None,
        judge_host: Optional[str] = None,
    ):
        """
        Initialize the smart router.

        Args:
            local_host: URL of local Ollama server
            remote_host: URL of remote SGLang server
            local_model: Model name for local Ollama
            remote_model: Model name for remote SGLang
            judge_model: Model for LLM-based classification (default: same as local_model)
            judge_host: Host for judge model (default: same as local_host)
        """
        self.local_client = ollama.Client(host=local_host)
        self.remote_client = ollama.Client(host=remote_host)
        self.local_model = local_model
        self.remote_model = remote_model

        # Judge model configuration
        self.judge_model = judge_model or local_model
        self.judge_host = judge_host or local_host
        self.judge_client = ollama.Client(host=self.judge_host)
```
**EN:** This block implements the initializer `__init__(local_host, remote_host, local_model, remote_model, judge_model, ...)` for `SmartRouter`. It prepares the object state and connects the instance to the surrounding smart router workflow.
**CN:** 该代码块实现 `SmartRouter` 的初始化方法 `__init__(local_host, remote_host, local_model, remote_model, judge_model, ...)`。它负责准备对象状态，并把实例接入 smart 路由器 相关的运行流程。

### Lines 69-102: Implement classify with llm / 实现classify with llm
```python
    def _classify_with_llm(
        self, prompt: str, verbose: bool = False
    ) -> tuple[bool, str]:
        """
        Use LLM to classify the prompt.

        Returns:
            Tuple of (use_remote, reason)
        """
        try:
            classification_prompt = self.CLASSIFICATION_PROMPT.format(
                prompt=prompt[:500]  # Limit prompt length for classification
            )

            response = self.judge_client.chat(
                model=self.judge_model,
                messages=[{"role": "user", "content": classification_prompt}],
                options={"temperature": 0, "num_predict": 10},
            )

            result = response["message"]["content"].strip().upper()

            if verbose:
                print(f"[Router] LLM Judge: {result}")

            if "COMPLEX" in result:
                return True, "Complex task"
            else:
                return False, "Simple task"

        except Exception as e:
            if verbose:
                print(f"[Router] LLM Judge failed: {e}, defaulting to local")
            return False, "Judge failed, defaulting to local"
```
**EN:** This block implements the method `_classify_with_llm(prompt, verbose)` on `SmartRouter`. It focuses on Use LLM to classify the prompt., so the class can advance the smart router workflow in a self-contained way.
**CN:** 该代码块实现 `SmartRouter` 上的方法 `_classify_with_llm(prompt, verbose)`。它围绕 `_classify_with_llm` 所承担的 smart 路由器 相关职责展开，使该类能够独立推进相应流程。

### Lines 104-115: Implement should use remote / 实现should use remote
```python
    def should_use_remote(self, prompt: str, verbose: bool = False) -> tuple[bool, str]:
        """
        Determine if the prompt should be routed to remote SGLang.

        Args:
            prompt: User's input prompt
            verbose: Print debug information

        Returns:
            Tuple of (should_use_remote, reason)
        """
        return self._classify_with_llm(prompt, verbose)
```
**EN:** This block implements the method `should_use_remote(prompt, verbose)` on `SmartRouter`. It focuses on Determine if the prompt should be routed to remote SGLang., so the class can advance the smart router workflow in a self-contained way.
**CN:** 该代码块实现 `SmartRouter` 上的方法 `should_use_remote(prompt, verbose)`。它围绕 `should_use_remote` 所承担的 smart 路由器 相关职责展开，使该类能够独立推进相应流程。

### Lines 117-152: Implement chat / 实现聊天
```python
    def chat(
        self,
        prompt: str,
        messages: Optional[list] = None,
        verbose: bool = False,
        force_local: bool = False,
        force_remote: bool = False,
    ) -> dict:
        """
        Route the request and get response.

        Args:
            prompt: User's input (used if messages is None)
            messages: Full message history (overrides prompt if provided)
            verbose: Print routing decision
            force_local: Force use of local model
            force_remote: Force use of remote model

        Returns:
            Response dict with 'content', 'model', 'location', 'reason' keys
        """
        # Build messages
        if messages is None:
            messages = [{"role": "user", "content": prompt}]
            check_prompt = prompt
        else:
            # Use the last user message for routing decision
            check_prompt = ""
            for msg in reversed(messages):
                if msg.get("role") == "user":
                    check_prompt = msg.get("content", "")
                    break

        # Determine routing
        if force_remote:
            use_remote, reason = True, "Forced remote"
```
**EN:** This block implements the method `chat(prompt, messages, verbose, force_local, force_remote)` on `SmartRouter`. It focuses on Route the request and get response., so the class can advance the smart router workflow in a self-contained way.
**CN:** 该代码块实现 `SmartRouter` 上的方法 `chat(prompt, messages, verbose, force_local, force_remote)`。它围绕 `chat` 所承担的 smart 路由器 相关职责展开，使该类能够独立推进相应流程。

### Lines 153-188: Continue chat / 继续说明聊天
```python
        elif force_local:
            use_remote, reason = False, "Forced local"
        else:
            use_remote, reason = self.should_use_remote(check_prompt, verbose)

        if use_remote:
            client = self.remote_client
            model = self.remote_model
            location = "Remote SGLang"
        else:
            client = self.local_client
            model = self.local_model
            location = "Local Ollama"

        if verbose:
            print(f"[Router] -> {location} | Model: {model}")

        try:
            response = client.chat(model=model, messages=messages)
            return {
                "content": response["message"]["content"],
                "model": model,
                "location": location,
                "reason": reason,
            }
        except Exception as e:
            # Fallback to the other option
            if verbose:
                print(f"[Router] {location} failed: {e}, falling back...")

            fallback_client = (
                self.remote_client if not use_remote else self.local_client
            )
            fallback_model = self.remote_model if not use_remote else self.local_model
            fallback_location = "Remote SGLang" if not use_remote else "Local Ollama"
```
**EN:** This block implements the method `chat(prompt, messages, verbose, force_local, force_remote)` on `SmartRouter`. It focuses on Route the request and get response., so the class can advance the smart router workflow in a self-contained way.
**CN:** 该代码块实现 `SmartRouter` 上的方法 `chat(prompt, messages, verbose, force_local, force_remote)`。它围绕 `chat` 所承担的 smart 路由器 相关职责展开，使该类能够独立推进相应流程。

### Lines 189-195: Continue chat / 继续说明聊天
```python
            response = fallback_client.chat(model=fallback_model, messages=messages)
            return {
                "content": response["message"]["content"],
                "model": fallback_model,
                "location": fallback_location,
                "reason": f"Fallback from {location}",
            }
```
**EN:** This block implements the method `chat(prompt, messages, verbose, force_local, force_remote)` on `SmartRouter`. It focuses on Route the request and get response., so the class can advance the smart router workflow in a self-contained way.
**CN:** 该代码块实现 `SmartRouter` 上的方法 `chat(prompt, messages, verbose, force_local, force_remote)`。它围绕 `chat` 所承担的 smart 路由器 相关职责展开，使该类能够独立推进相应流程。

### Lines 197-241: Implement chat stream / 实现聊天 流式
```python
    def chat_stream(
        self,
        prompt: str,
        messages: Optional[list] = None,
        verbose: bool = False,
        force_local: bool = False,
        force_remote: bool = False,
    ):
        """
        Route the request and stream response.

        Yields:
            Response chunks
        """
        if messages is None:
            messages = [{"role": "user", "content": prompt}]
            check_prompt = prompt
        else:
            check_prompt = ""
            for msg in reversed(messages):
                if msg.get("role") == "user":
                    check_prompt = msg.get("content", "")
                    break

        if force_remote:
            use_remote, reason = True, "Forced remote"
        elif force_local:
            use_remote, reason = False, "Forced local"
        else:
            use_remote, reason = self.should_use_remote(check_prompt, verbose)

        if use_remote:
            client = self.remote_client
            model = self.remote_model
            location = "Remote SGLang"
        else:
            client = self.local_client
            model = self.local_model
            location = "Local Ollama"

        if verbose:
            print(f"[Router] -> {location} | Model: {model}")

        for chunk in client.chat(model=model, messages=messages, stream=True):
            yield chunk
```
**EN:** This block implements the method `chat_stream(prompt, messages, verbose, force_local, force_remote)` on `SmartRouter`. It focuses on Route the request and stream response., so the class can advance the smart router workflow in a self-contained way.
**CN:** 该代码块实现 `SmartRouter` 上的方法 `chat_stream(prompt, messages, verbose, force_local, force_remote)`。它围绕 `chat_stream` 所承担的 smart 路由器 相关职责展开，使该类能够独立推进相应流程。

### Lines 244-279: Implement main / 实现main
```python
def main():
    """Interactive demo of the smart router."""
    print("=" * 60)
    print("Smart Router: Local Ollama <-> Remote SGLang")
    print("=" * 60)
    print("\nRouting strategy:")
    print("  LLM Judge classifies each request as SIMPLE or COMPLEX")
    print("  - SIMPLE tasks -> Local Ollama (fast)")
    print("  - COMPLEX tasks -> Remote SGLang (powerful)")
    print("\nType 'quit' to exit\n")

    router = SmartRouter(
        local_host="http://localhost:11434",
        remote_host="http://localhost:30001",
        local_model="llama3.2",
        remote_model="Qwen/Qwen2.5-1.5B-Instruct",
    )

    messages = []
    while True:
        try:
            user_input = input("You: ").strip()
            if user_input.lower() in ["quit", "exit", "q"]:
                print("Goodbye!")
                break
            if not user_input:
                continue

            messages.append({"role": "user", "content": user_input})

            # Use streaming for real-time output
            print("\nAssistant: ", end="", flush=True)
            full_response = ""
            for chunk in router.chat_stream(
                prompt=user_input, messages=messages, verbose=True
            ):
```
**EN:** This block implements the function `main()`. It focuses on Interactive demo of the smart router., providing reusable behavior for the smart router pipeline.
**CN:** 该代码块实现函数 `main()`。它围绕 `main` 所承担的 smart 路由器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 280-292: Continue main / 继续说明main
```python
                content = chunk.get("message", {}).get("content", "")
                if content:
                    print(content, end="", flush=True)
                    full_response += content
            print("\n")

            messages.append({"role": "assistant", "content": full_response})

        except KeyboardInterrupt:
            print("\nGoodbye!")
            break
        except Exception as e:
            print(f"Error: {e}\n")
```
**EN:** This block implements the function `main()`. It focuses on Interactive demo of the smart router., providing reusable behavior for the smart router pipeline.
**CN:** 该代码块实现函数 `main()`。它围绕 `main` 所承担的 smart 路由器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 295-296: Run the module as a script / 作为脚本运行模块
```python
if __name__ == "__main__":
    main()
```
**EN:** This block provides the executable entry path used when the module is launched directly.
**CN:** 该代码块提供了模块被直接运行时使用的执行入口。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SmartRouter
- **Main callables / 主要可调用对象**: main
- **Domain focus / 领域焦点**: smart router / smart 路由器
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: typing
- **Third-party / 第三方库**: ollama
- **Local Modules / 本地模块**: None / 无

# exceptions.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `vllm/exceptions.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines a small set of vLLM-specific exception types for validation and missing-resource errors. / 定义少量 vLLM 专用异常类型，用于参数校验和资源缺失场景。

## Line-by-Line Analysis / 逐行分析
### Validation error with contextual metadata (lines 9-36)
```python
class VLLMValidationError(ValueError):
    """vLLM-specific validation error for request validation failures.

    Args:
        message: The error message describing the validation failure.
        parameter: Optional parameter name that failed validation.
        value: Optional value that was rejected during validation.
    """

    def __init__(
        self,
        message: str,
        *,
        parameter: str | None = None,
        value: Any = None,
    ) -> None:
        super().__init__(message)
        self.parameter = parameter
        self.value = value

    def __str__(self):
        base = super().__str__()
        extras = []
        if self.parameter is not None:
            extras.append(f"parameter={self.parameter}")
        if self.value is not None:
            extras.append(f"value={self.value}")
        return f"{base} ({', '.join(extras)})" if extras else base
```
**EN:** `VLLMValidationError` subclasses `ValueError` but adds optional `parameter` and `value` fields. Its custom `__str__()` appends those fields only when present, which keeps normal error messages clean while still making debugging output precise enough to identify the rejected input.
**CN:** `VLLMValidationError` 继承自 `ValueError`，但额外记录可选的 `parameter` 和 `value` 字段。它的 `__str__()` 只在这些字段存在时附加到消息末尾，因此既能保持普通报错简洁，又能在调试时精确指出被拒绝的输入。

### Not-found hierarchy for adapters (lines 39-66)
```python
class VLLMNotFoundError(Exception):
    """vLLM-specific NotFoundError"""

    pass


class LoRAAdapterNotFoundError(VLLMNotFoundError):
    """Exception raised when a LoRA adapter is not found.

    This exception is thrown when a requested LoRA adapter does not exist
    in the system.

    Attributes:
        message: The error message string describing the exception
    """

    message: str

    def __init__(
        self,
        lora_name: str,
        lora_path: str,
    ) -> None:
        message = f"Loading lora {lora_name} failed: No adapter found for {lora_path}"
        self.message = message

    def __str__(self):
        return self.message
```
**EN:** `VLLMNotFoundError` provides a namespace-level base class, and `LoRAAdapterNotFoundError` specializes it with a deterministic message containing both the requested adapter name and the lookup path. That makes higher layers able to catch generic not-found conditions or LoRA-specific failures separately.
**CN:** `VLLMNotFoundError` 提供了一个命名空间级别的基类，而 `LoRAAdapterNotFoundError` 在其上进一步细化，生成同时包含适配器名和查找路径的确定性错误消息。这样上层代码既可以统一捕获“未找到”类错误，也可以专门处理 LoRA 适配器缺失。

## Key Concepts / 关键概念
- EN: vLLM prefers semantically named exceptions over raw `ValueError`/`Exception` when an error has API-level meaning.
- CN: 当错误具有 API 级语义时，vLLM 更倾向于使用语义化命名的异常，而不是直接抛出原始 `ValueError`/`Exception`。
- EN: Attaching rejected parameter/value data is useful for request validation paths where user input is the root cause.
- CN: 在请求校验场景中，附带被拒绝的参数和值有助于快速定位问题根因。

## Dependencies / 依赖关系
- EN: Only depends on `typing.Any`; there are no external runtime dependencies.
- CN: 仅依赖 `typing.Any`，没有外部运行时依赖。
- EN: Likely consumed by request-validation, LoRA-loading, and API error translation layers elsewhere in the codebase.
- CN: 通常会被请求校验、LoRA 加载以及 API 错误转换层消费。

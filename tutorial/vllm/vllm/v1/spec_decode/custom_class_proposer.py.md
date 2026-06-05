# custom_class_proposer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/spec_decode/custom_class_proposer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `create_custom_proposer` for the V1 `spec_decode` subsystem. / 为 V1 的 `spec_decode` 子系统实现 `create_custom_proposer`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import importlib

from vllm.config import VllmConfig
from vllm.logger import init_logger

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `create_custom_proposer` function / `create_custom_proposer` 函数
```python
def create_custom_proposer(vllm_config: VllmConfig):
    """Load and instantiate a user-provided proposer class.

    The class path is read from ``speculative_config.model``
    (e.g., ``"my_module.MyCustomProposer"``).  The class is
    imported, instantiated with *vllm_config*, and returned
    directly so the caller can use it without any wrapper.

    The returned object must expose a callable ``propose`` method.
    """
    assert vllm_config.speculative_config is not None
    spec_config = vllm_config.speculative_config

    backend = spec_config.model
    assert backend is not None

    if "." not in backend:
        raise ValueError(
            f"Invalid custom proposer module path '{backend}'. "
            "It must be a full module path (e.g., 'module.MyProposerClass')."
        )

    module_path, class_name = backend.rsplit(".", 1)
    try:
        module = importlib.import_module(module_path)
    except ImportError as e:
        raise ImportError(
            f"Cannot import module '{module_path}' for custom proposer '{backend}': {e}"
        ) from e

    user_class = getattr(module, class_name, None)
    if user_class is None:
        raise AttributeError(
            f"Module '{module_path}' has no attribute '{class_name}' "
            f"(speculative_config.model='{backend}')"
        )

    try:
        instance = user_class(vllm_config)
    except Exception as e:
        raise RuntimeError(
            f"Failed to instantiate custom proposer class '{backend}': {e}. "
            "The class constructor must accept VllmConfig as argument."
        ) from e

    if not hasattr(instance, "propose"):
        raise AttributeError(
            f"Custom proposer class '{backend}' must have a 'propose' method."
        )
    if not callable(instance.propose):
        raise AttributeError(
            f"Custom proposer class '{backend}' has a 'propose' attribute "
            "but it is not callable."
        )

    logger.info(
        "Loaded custom proposer class '%s' with num_speculative_tokens=%d",
        backend,
        spec_config.num_speculative_tokens,
    )

    return instance
```
**EN:** This function creates a new object or plan within the module. The docstring frames it as: Load and instantiate a user-provided proposer class. Key calls include `rsplit`, `getattr`, `info`, `ValueError`, `import_module`, `AttributeError`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会创建新的对象或计划，其作用域位于the module。 关键调用包括 `rsplit`, `getattr`, `info`, `ValueError`, `import_module`, `AttributeError`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `create_custom_proposer`: top-level helper or orchestration entry point. / `create_custom_proposer`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `importlib`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.logger`

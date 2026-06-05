# punica_selector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/lora/punica_wrapper/punica_selector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Resolve and instantiate the correct Punica wrapper for the active platform. / [CN] 为当前运行平台解析并实例化正确的 Punica wrapper 实现。

## Line-by-Line Analysis / 逐行分析
### Runtime dependencies / 运行时依赖
```python
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.utils.import_utils import resolve_obj_by_qualname

from .punica_base import PunicaWrapperBase

logger = init_logger(__name__)
```
**EN:** The module depends on platform introspection, dynamic import utilities, and a typed base class. A module-level logger records the selected backend once.
**CN:** 模块依赖平台探测、动态导入工具以及类型化基类。模块级 logger 用于记录一次被选中的后端实现。

### Factory function / 工厂函数
```python
def get_punica_wrapper(*args, **kwargs) -> PunicaWrapperBase:
    punica_wrapper_qualname = current_platform.get_punica_wrapper()
    punica_wrapper_cls = resolve_obj_by_qualname(punica_wrapper_qualname)
    punica_wrapper = punica_wrapper_cls(*args, **kwargs)
    assert punica_wrapper is not None, (
        "the punica_wrapper_qualname(" + punica_wrapper_qualname + ") is wrong."
    )
    logger.info_once("Using %s.", punica_wrapper_qualname.rsplit(".", 1)[1])
    return punica_wrapper
```
**EN:** `current_platform` provides a fully qualified class name, `resolve_obj_by_qualname` imports it, and the function instantiates the wrapper with transparent passthrough arguments. The assertion protects against bad configuration, and `info_once` avoids repetitive logs.
**CN:** `current_platform` 给出完整类名，`resolve_obj_by_qualname` 负责导入，随后函数把参数原样透传给构造器完成实例化。断言用于防止配置错误，`info_once` 则避免重复打印日志。

## Key Concepts / 关键概念
- Dynamic backend selection. / 动态后端选择。
- Platform abstraction through `current_platform`. / 通过 `current_platform` 屏蔽平台差异。
- Late import by qualified name. / 按限定名延迟导入对象。

## Dependencies / 依赖关系
- `vllm.platforms.current_platform`
- `vllm.utils.import_utils.resolve_obj_by_qualname`
- `vllm.logger.init_logger`
- `PunicaWrapperBase`

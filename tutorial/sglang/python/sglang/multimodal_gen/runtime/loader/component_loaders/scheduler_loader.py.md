# scheduler_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/loader/component_loaders/scheduler_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the component loading layer. It centers on `SchedulerLoader`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于组件加载层。它围绕 `SchedulerLoader` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: module setup and imports / 模块初始化与导入
```python
from sglang.multimodal_gen.runtime.loader.component_loaders.component_loader import (
    ComponentLoader,
)
from sglang.multimodal_gen.runtime.models.registry import ModelRegistry
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.hf_diffusers_utils import (
    get_diffusers_component_config,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.models.registry`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, and `sglang.multimodal_gen.runtime.utils.logging_utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`、`sglang.multimodal_gen.runtime.models.registry`、`sglang.multimodal_gen.runtime.server_args`、`sglang.multimodal_gen.runtime.utils.hf_diffusers_utils` 和 `sglang.multimodal_gen.runtime.utils.logging_utils`。这些依赖为后续实现提供所需符号。

### Lines 11-11: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 14-16: `SchedulerLoader` class overview / `SchedulerLoader` 类概览
```python
class SchedulerLoader(ComponentLoader):
    """Loader for scheduler."""
```
**EN:** This block defines class `SchedulerLoader`. Loader for scheduler. It inherits from `ComponentLoader`.
**CN:** 该代码块定义了类 `SchedulerLoader`。 它用于封装 scheduler loader 相关行为。 它继承自 `ComponentLoader`。

### Lines 17-18: supporting statements / 辅助语句
```python
    component_names = ["scheduler"]
    expected_library = "diffusers"
```
**EN:** This block gathers supporting statements inside `SchedulerLoader`. It updates names such as `component_names`, and `expected_library`.
**CN:** 该代码块汇集了位于 `SchedulerLoader` 内部的辅助语句。 它会更新 `component_names` 和 `expected_library` 等名称。

### Lines 20-37: `load_customized` implementation / `load_customized` 实现
```python
    def load_customized(
        self, component_model_path: str, server_args: ServerArgs, *args
    ):
        """Load the scheduler based on the model path, and inference args."""
        config = get_diffusers_component_config(component_path=component_model_path)

        class_name = config.pop("_class_name")
        assert (
            class_name is not None
        ), "Model config does not contain a _class_name attribute. Only diffusers format is supported."

        scheduler_cls, _ = ModelRegistry.resolve_model_cls(class_name)

        scheduler = scheduler_cls(**config)
        if server_args.pipeline_config.flow_shift is not None:
            scheduler.set_shift(server_args.pipeline_config.flow_shift)

        return scheduler
```
**EN:** This block defines method `load_customized` on `SchedulerLoader`. Load the scheduler based on the model path, and inference args. Key calls include `get_diffusers_component_config`, `config.pop`, `ModelRegistry.resolve_model_cls`, `scheduler_cls`, and `scheduler.set_shift`. The implementation branches on conditions. Parameters such as `component_model_path`, and `server_args` drive the behavior in this section.
**CN:** 该代码块定义了 `SchedulerLoader` 的方法 `load_customized`。 它用于加载customized。 关键调用包括 `get_diffusers_component_config`、`config.pop`、`ModelRegistry.resolve_model_cls`、`scheduler_cls` 和 `scheduler.set_shift`。 实现中包含条件分支。 本段逻辑主要由 `component_model_path` 和 `server_args` 等参数驱动。

## Key Concepts / 关键概念
- `SchedulerLoader`: Loader for scheduler. / 核心类，用于封装 scheduler loader 相关行为。

## Dependencies / 依赖关系
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.loader.component_loaders.component_loader`, `sglang.multimodal_gen.runtime.models.registry`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 37

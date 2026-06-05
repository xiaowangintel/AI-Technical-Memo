# generator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/apps/ComfyUI_SGLDiffusion/core/generator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the ComfyUI integration layer. It centers on `SGLDiffusionGenerator`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Generator for SGLang Diffusion ComfyUI integration. / 该文件属于ComfyUI 集成层。它围绕 `SGLDiffusionGenerator` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module setup and imports / 模块初始化与导入
```python
"""
Generator for SGLang Diffusion ComfyUI integration.
"""

import logging
import os

import psutil
from comfy import model_detection, model_management
from comfy.utils import (
    calculate_parameters,
    load_torch_file,
    state_dict_prefix_replace,
    unet_to_diffusers,
)
```
**EN:** This block establishes the module context and imports `logging`, `os`, `psutil`, `comfy`, and `comfy.utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `logging`、`os`、`psutil`、`comfy` 和 `comfy.utils`。这些依赖为后续实现提供所需符号。

### Lines 17-32: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)

try:
    from sglang.multimodal_gen import DiffGenerator
except ImportError:
    logger.error(
        "Error: sglang.multimodal_gen is not installed. Please install it using 'pip install sglang[diffusion]'"
    )

from ..executors import (
    FluxExecutor,
    QwenImageEditExecutor,
    QwenImageExecutor,
    ZImageExecutor,
)
from .model_patcher import SGLDModelPatcher
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`, and `logger.error`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 和 `logger.error` 协同工作。

### Lines 35-37: `SGLDiffusionGenerator` class overview / `SGLDiffusionGenerator` 类概览
```python
class SGLDiffusionGenerator:
    """Generator for SGLang Diffusion models in ComfyUI."""
```
**EN:** This block defines class `SGLDiffusionGenerator`. Generator for SGLang Diffusion models in ComfyUI.
**CN:** 该代码块定义了类 `SGLDiffusionGenerator`。 它用于封装 sgldiffusion generator 相关行为。

### Lines 38-55: `__init__` implementation / `__init__` 实现
```python
    def __init__(self):
        self.model_path = None
        self.generator = None
        self.executor = None
        self.last_options = None

        self.pipeline_class_dict = {
            "flux": "ComfyUIFluxPipeline",
            "lumina2": "ComfyUIZImagePipeline",  # zimage
            "qwen_image": "ComfyUIQwenImagePipeline",
            "qwen_image_edit": "ComfyUIQwenImageEditPipeline",
        }
        self.executor_class_dict = {
            "flux": FluxExecutor,
            "lumina2": ZImageExecutor,
            "qwen_image": QwenImageExecutor,
            "qwen_image_edit": QwenImageEditExecutor,
        }
```
**EN:** This block defines method `__init__` on `SGLDiffusionGenerator`. It initializes the instance state.
**CN:** 该代码块定义了 `SGLDiffusionGenerator` 的方法 `__init__`。 它用于初始化实例状态。

### Lines 57-58: `__del__` implementation / `__del__` 实现
```python
    def __del__(self):
        self.close_generator()
```
**EN:** This block defines method `__del__` on `SGLDiffusionGenerator`. It releases held resources during teardown. Key calls include `self.close_generator`.
**CN:** 该代码块定义了 `SGLDiffusionGenerator` 的方法 `__del__`。 它用于在销毁阶段释放占用资源。 关键调用包括 `self.close_generator`。

### Lines 60-75: `init_generator` implementation / `init_generator` 实现
```python
    def init_generator(
        self, model_path: str, pipeline_class_name: str, kwargs: dict = None
    ):
        """Initialize the diffusion generator."""
        if self.generator is not None:
            return self.generator
        if kwargs is None:
            kwargs = {}
        # Set comfyui_mode for ComfyUI integration
        kwargs["comfyui_mode"] = True
        self.generator = DiffGenerator.from_pretrained(
            model_path=model_path,
            pipeline_class_name=pipeline_class_name,
            **kwargs,
        )
        return self.generator
```
**EN:** This block defines method `init_generator` on `SGLDiffusionGenerator`. Initialize the diffusion generator. Key calls include `DiffGenerator.from_pretrained`. The implementation branches on conditions. Parameters such as `model_path`, `pipeline_class_name`, and `kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionGenerator` 的方法 `init_generator`。 它用于初始化generator。 关键调用包括 `DiffGenerator.from_pretrained`。 实现中包含条件分支。 本段逻辑主要由 `model_path`、`pipeline_class_name` 和 `kwargs` 等参数驱动。

### Lines 77-113: `kill_generator` implementation / `kill_generator` 实现
```python
    def kill_generator(self):
        """Kill worker processes manually because generator shutdown cannot terminate them."""
        current_pid = os.getpid()
        worker_processes = []
        for proc in psutil.process_iter(["pid", "name", "cmdline"]):
            try:
                # Look for sglang-diffusionWorker processes
                if proc.info["cmdline"]:
                    cmdline = " ".join(proc.info["cmdline"])
                    if "sgl_diffusion::" in cmdline:
                        if proc.info["pid"] != current_pid:
                            worker_processes.append(proc)
            except (psutil.NoSuchProcess, psutil.AccessDenied):
                continue

        if worker_processes:
            logger.info(
                f"Found {len(worker_processes)} worker processes to terminate..."
            )
            for proc in worker_processes:
                try:
                    logger.info(
                        f"Terminating worker process {proc.info['pid']}: {proc.info['name']}"
                    )
                    proc.terminate()
                    proc.wait(timeout=5)
                except psutil.TimeoutExpired:
                    logger.warning(
                        f"Process {proc.info['pid']} did not terminate, forcing kill..."
                    )
                    try:
                        proc.kill()
                        proc.wait(timeout=2)
                    except (psutil.NoSuchProcess, psutil.TimeoutExpired):
                        pass
                except (psutil.NoSuchProcess, psutil.AccessDenied):
                    pass
```
**EN:** This block defines method `kill_generator` on `SGLDiffusionGenerator`. Kill worker processes manually because generator shutdown cannot terminate them. Key calls include `os.getpid`, `psutil.process_iter`, `logger.info`, `join`, and `proc.terminate`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths.
**CN:** 该代码块定义了 `SGLDiffusionGenerator` 的方法 `kill_generator`。 它用于终止generator。 关键调用包括 `os.getpid`、`psutil.process_iter`、`logger.info`、`join` 和 `proc.terminate`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。

### Lines 115-124: `close_generator` implementation / `close_generator` 实现
```python
    def close_generator(self):
        """Close and cleanup the generator and all associated resources."""
        if self.generator is not None:
            self.generator.shutdown()
            self.kill_generator()
            # Clear other references
            self.last_options = None
            self.model_path = None
            self.generator = None
            self.executor = None
```
**EN:** This block defines method `close_generator` on `SGLDiffusionGenerator`. Close and cleanup the generator and all associated resources. Key calls include `self.generator.shutdown`, and `self.kill_generator`. The implementation branches on conditions.
**CN:** 该代码块定义了 `SGLDiffusionGenerator` 的方法 `close_generator`。 它用于关闭generator。 关键调用包括 `self.generator.shutdown` 和 `self.kill_generator`。 实现中包含条件分支。

### Lines 126-183: `get_comfyui_model` implementation / `get_comfyui_model` 实现
```python
    def get_comfyui_model(self, model_path: str, model_options: dict = None):
        """Get ComfyUI model from model path."""
        if model_options is None:
            model_options = {}
        dtype = model_options.get("dtype", None)
        # Allow loading unets from checkpoint files
        sd = load_torch_file(model_path)
        diffusion_model_prefix = model_detection.unet_prefix_from_state_dict(sd)
        temp_sd = state_dict_prefix_replace(
            sd, {diffusion_model_prefix: ""}, filter_keys=True
        )
        if len(temp_sd) > 0:
            sd = temp_sd

        parameters = calculate_parameters(sd)
        load_device = model_management.get_torch_device()

        model_detect_config = model_detection.detect_unet_config(sd, "")
        model_type = model_detect_config.get("image_model", None)
        if model_type is None or model_type not in self.pipeline_class_dict:
            raise ValueError(f"Unsupported model type: {model_type}")
        model_config = model_detection.model_config_from_unet(sd, "")

        if model_config is not None:
            new_sd = sd
        else:
            new_sd = model_detection.convert_diffusers_mmdit(sd, "")
            if new_sd is not None:  # diffusers mmdit
                model_config = model_detection.model_config_from_unet(new_sd, "")
                if model_config is None:
                    return None
            else:  # diffusers unet
                model_config = model_detection.model_config_from_diffusers_unet(sd)
                if model_config is None:
                    return None

                diffusers_keys = unet_to_diffusers(model_config.unet_config)
                new_sd = {}
                for k in diffusers_keys:
                    if k in sd:
                        new_sd[diffusers_keys[k]] = sd.pop(k)
        offload_device = model_management.unet_offload_device()
        if dtype is None:
            unet_dtype = model_management.unet_dtype(
                model_params=parameters,
                supported_dtypes=model_config.supported_inference_dtypes,
            )
        else:
            unet_dtype = dtype

        manual_cast_dtype = model_management.unet_manual_cast(
            unet_dtype, load_device, model_config.supported_inference_dtypes
        )
        model_config.set_inference_dtype(unet_dtype, manual_cast_dtype)
        model_config.custom_operations = model_options.get("custom_operations", None)
        model_config.unet_config["disable_unet_model_creation"] = True
        comfyui_model = model_config.get_model({})
        return comfyui_model, model_config, model_type
```
**EN:** This block defines method `get_comfyui_model` on `SGLDiffusionGenerator`. Get ComfyUI model from model path. Key calls include `model_options.get`, `load_torch_file`, `model_detection.unet_prefix_from_state_dict`, `state_dict_prefix_replace`, and `calculate_parameters`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `model_path`, and `model_options` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionGenerator` 的方法 `get_comfyui_model`。 它用于获取comfyui model。 关键调用包括 `model_options.get`、`load_torch_file`、`model_detection.unet_prefix_from_state_dict`、`state_dict_prefix_replace` 和 `calculate_parameters`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `model_path` 和 `model_options` 等参数驱动。

### Lines 185-232: `load_model` implementation / `load_model` 实现
```python
    def load_model(
        self, model_path: str, model_options: dict = None, sgld_options: dict = None
    ):
        """Load model and return model patcher."""
        gather_options = {
            "model_path": model_path,
            "model_options": model_options,
            "sgld_options": sgld_options,
        }
        if (
            self.last_options is not None
            and self.last_options == gather_options
            and self.generator is not None
        ):
            return self.generator
        else:
            self.close_generator()

        self.last_options = gather_options
        self.model_path = model_path

        comfyui_model, model_config, model_type = self.get_comfyui_model(
            model_path, model_options
        )
        if model_type is None or model_type not in self.pipeline_class_dict:
            raise ValueError(f"Unsupported model type: {model_type}")

        set_model_type = sgld_options.pop("model_type", None) if sgld_options else None
        if set_model_type is not None and set_model_type in self.pipeline_class_dict:
            model_type = set_model_type

        pipeline_class_name = self.pipeline_class_dict[model_type]
        self.generator = self.init_generator(
            model_path, pipeline_class_name, sgld_options
        )

        executor_class = self.executor_class_dict[model_type]
        self.executor = executor_class(
            self.generator, model_path, comfyui_model, model_config
        )
        comfyui_model.diffusion_model = self.executor

        load_device = model_management.get_torch_device()
        offload_device = model_management.unet_offload_device()

        return SGLDModelPatcher(
            comfyui_model, load_device, offload_device, model_type=model_type
        )
```
**EN:** This block defines method `load_model` on `SGLDiffusionGenerator`. Load model and return model patcher. Key calls include `self.get_comfyui_model`, `self.init_generator`, `executor_class`, `model_management.get_torch_device`, and `model_management.unet_offload_device`. The implementation branches on conditions. Parameters such as `model_path`, `model_options`, and `sgld_options` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionGenerator` 的方法 `load_model`。 它用于加载model。 关键调用包括 `self.get_comfyui_model`、`self.init_generator`、`executor_class`、`model_management.get_torch_device` 和 `model_management.unet_offload_device`。 实现中包含条件分支。 本段逻辑主要由 `model_path`、`model_options` 和 `sgld_options` 等参数驱动。

## Key Concepts / 关键概念
- `SGLDiffusionGenerator`: Generator for SGLang Diffusion models in ComfyUI. / 核心类，用于封装 sgldiffusion generator 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `os`
- **Third-party / 第三方依赖**: `psutil`, `comfy`, `comfy.utils`
- **Internal modules / 内部模块**: `..executors`, `.model_patcher`, `sglang.multimodal_gen`

- **Total lines / 总行数**: 232

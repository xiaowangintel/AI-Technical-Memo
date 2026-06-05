# nodes.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/apps/ComfyUI_SGLDiffusion/nodes.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the ComfyUI integration layer. It centers on `SGLDOptions`, `SGLDLoraLoader`, and `SGLDUNETLoader`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: ComfyUI nodes for SGLang Diffusion integration. Provides nodes for connecting to SGLang Diffusion server and generating images/videos. / 该文件属于ComfyUI 集成层。它围绕 `SGLDOptions`、`SGLDLoraLoader` 和 `SGLDUNETLoader` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module setup and imports / 模块初始化与导入
```python
"""
ComfyUI nodes for SGLang Diffusion integration.
Provides nodes for connecting to SGLang Diffusion server and generating images/videos.
"""

import os
import uuid

import folder_paths
import torch

from .core import SGLDiffusionGenerator, SGLDiffusionServerAPI
from .utils import (
    convert_b64_to_tensor_image,
    convert_video_to_comfy_video,
    get_image_path,
    is_empty_image,
)
```
**EN:** This block establishes the module context and imports `os`, `uuid`, `folder_paths`, `torch`, `.core`, and `.utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `os`、`uuid`、`folder_paths`、`torch`、`.core` 和 `.utils`。这些依赖为后续实现提供所需符号。

### Lines 21-21: `SGLDOptions` class overview / `SGLDOptions` 类概览
```python
class SGLDOptions:
```
**EN:** This block defines class `SGLDOptions`. It encapsulates sgldoptions behavior.
**CN:** 该代码块定义了类 `SGLDOptions`。 它用于封装 sgldoptions 相关行为。

### Lines 22-65: `INPUT_TYPES` implementation / `INPUT_TYPES` 实现
```python
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {},
            "optional": {
                "model_type": (
                    ["auto-detect", "qwen_image", "qwen_image_edit", "flux", "lumina2"],
                    {"default": "auto-detect"},
                ),
                "enable_torch_compile": (
                    "BOOLEAN",
                    {"default": False},
                ),
                "num_gpus": ("INT", {"default": 1, "min": 1, "step": 1}),
                "tp_size": ("INT", {"default": -1, "min": -1, "step": 1}),
                "sp_degree": ("INT", {"default": -1, "min": -1, "step": 1}),
                "ulysses_degree": (
                    "INT",
                    {
                        "default": -1,
                        "min": -1,
                        "step": 1,
                    },
                ),
                "ring_degree": (
                    "INT",
                    {
                        "default": -1,
                        "min": -1,
                        "step": 1,
                    },
                ),
                "dp_size": ("INT", {"default": 1, "min": 1, "step": 1}),
                "dp_degree": ("INT", {"default": 1, "min": 1, "step": 1}),
                "enable_cfg_parallel": (
                    "BOOLEAN",
                    {"default": False},
                ),
                "attention_backend": (
                    "STRING",
                    {"default": ""},
                ),
            },
        }
```
**EN:** This block defines method `INPUT_TYPES` on `SGLDOptions`. It handles input types logic.
**CN:** 该代码块定义了 `SGLDOptions` 的方法 `INPUT_TYPES`。 它用于处理 input types 相关逻辑。

### Lines 67-70: supporting statements / 辅助语句
```python
    RETURN_TYPES = ("SGLD_OPTIONS",)
    RETURN_NAMES = ("sgld_options",)
    FUNCTION = "create_options"
    CATEGORY = "SGLDiffusion"
```
**EN:** This block gathers supporting statements inside `SGLDOptions`. It updates names such as `RETURN_TYPES`, `RETURN_NAMES`, `FUNCTION`, and `CATEGORY`.
**CN:** 该代码块汇集了位于 `SGLDOptions` 内部的辅助语句。 它会更新 `RETURN_TYPES`、`RETURN_NAMES`、`FUNCTION` 和 `CATEGORY` 等名称。

### Lines 72-110: `create_options` implementation / `create_options` 实现
```python
    def create_options(
        self,
        model_type: str = "auto-detect",
        enable_torch_compile: bool = False,
        num_gpus: int = 1,
        tp_size: int = -1,
        sp_degree: int = -1,
        ulysses_degree: int = -1,
        ring_degree: int = -1,
        dp_size: int = 1,
        dp_degree: int = 1,
        enable_cfg_parallel: bool = False,
        attention_backend: str = "",
    ):
        """
        Build a dictionary of SGLang Diffusion runtime options.
        """
        # Convert -1 to None for optional parameters (matching ServerArgs defaults)
        ulysses_degree = None if ulysses_degree == -1 else ulysses_degree
        ring_degree = None if ring_degree == -1 else ring_degree
        attention_backend = None if attention_backend == "" else attention_backend

        options = {
            "model_type": model_type,
            "enable_torch_compile": enable_torch_compile,
            "num_gpus": num_gpus,
            "tp_size": tp_size,
            "sp_degree": sp_degree,
            "ulysses_degree": ulysses_degree,
            "ring_degree": ring_degree,
            "dp_size": dp_size,
            "dp_degree": dp_degree,
            "enable_cfg_parallel": enable_cfg_parallel,
            "attention_backend": attention_backend,
        }

        # Strip None to keep payload clean
        options = {k: v for k, v in options.items() if v is not None}
        return (options,)
```
**EN:** This block defines method `create_options` on `SGLDOptions`. Build a dictionary of SGLang Diffusion runtime options. Key calls include `options.items`. Parameters such as `model_type`, `enable_torch_compile`, `num_gpus`, `tp_size`, and `sp_degree` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDOptions` 的方法 `create_options`。 它用于创建options。 关键调用包括 `options.items`。 本段逻辑主要由 `model_type`、`enable_torch_compile`、`num_gpus`、`tp_size` 和 `sp_degree` 等参数驱动。

### Lines 113-113: `SGLDLoraLoader` class overview / `SGLDLoraLoader` 类概览
```python
class SGLDLoraLoader:
```
**EN:** This block defines class `SGLDLoraLoader`. It encapsulates sgldlora loader behavior.
**CN:** 该代码块定义了类 `SGLDLoraLoader`。 它用于封装 sgldlora loader 相关行为。

### Lines 114-130: `INPUT_TYPES` implementation / `INPUT_TYPES` 实现
```python
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "model": ("MODEL",),
                "lora_name": (folder_paths.get_filename_list("loras"),),
                "strength_model": (
                    "FLOAT",
                    {"default": 1.0, "min": 0, "max": 10, "step": 0.01},
                ),
                "nickname": ("STRING", {"default": ""}),
                "target": (
                    ["all", "transformer", "transformer_2", "critic"],
                    {"default": "all"},
                ),
            },
        }
```
**EN:** This block defines method `INPUT_TYPES` on `SGLDLoraLoader`. It handles input types logic. Key calls include `folder_paths.get_filename_list`.
**CN:** 该代码块定义了 `SGLDLoraLoader` 的方法 `INPUT_TYPES`。 它用于处理 input types 相关逻辑。 关键调用包括 `folder_paths.get_filename_list`。

### Lines 132-135: supporting statements / 辅助语句
```python
    RETURN_TYPES = ("MODEL",)
    FUNCTION = "load_lora"

    CATEGORY = "SGLDiffusion"
```
**EN:** This block gathers supporting statements inside `SGLDLoraLoader`. It updates names such as `RETURN_TYPES`, `FUNCTION`, and `CATEGORY`.
**CN:** 该代码块汇集了位于 `SGLDLoraLoader` 内部的辅助语句。 它会更新 `RETURN_TYPES`、`FUNCTION` 和 `CATEGORY` 等名称。

### Lines 137-163: `load_lora` implementation / `load_lora` 实现
```python
    def load_lora(
        self, model, lora_name, strength_model=1.0, nickname="", target="all"
    ):
        """Load LoRA adapter using SGLang Diffusion API."""
        lora_path = folder_paths.get_full_path("loras", lora_name)
        assert model is not None
        bi = model.clone()
        nickname = nickname if nickname != "" else str("lora" + str(uuid.uuid4()))
        # set lora in the model
        bi.patches[nickname] = (lora_path, strength_model, target)

        # prepare input for the SGLang Diffusion API
        lora_input = {
            "lora_nickname": [],
            "lora_path": [],
            "strength": [],
            "target": [],
        }
        for nickname, lora_info in bi.patches.items():
            lora_input["lora_nickname"].append(nickname)
            lora_input["lora_path"].append(lora_info[0])
            lora_input["strength"].append(lora_info[1])
            lora_input["target"].append(lora_info[2])

        # call the SGLang Diffusion API
        model.model.diffusion_model.set_lora(**lora_input)
        return (model,)
```
**EN:** This block defines method `load_lora` on `SGLDLoraLoader`. Load LoRA adapter using SGLang Diffusion API. Key calls include `folder_paths.get_full_path`, `model.clone`, `bi.patches.items`, `model.model.diffusion_model.set_lora`, and `str`. The implementation iterates over collections or steps. Parameters such as `model`, `lora_name`, `strength_model`, `nickname`, and `target` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDLoraLoader` 的方法 `load_lora`。 它用于加载lora。 关键调用包括 `folder_paths.get_full_path`、`model.clone`、`bi.patches.items`、`model.model.diffusion_model.set_lora` 和 `str`。 实现中会遍历集合或步骤。 本段逻辑主要由 `model`、`lora_name`、`strength_model`、`nickname` 和 `target` 等参数驱动。

### Lines 166-166: `SGLDUNETLoader` class overview / `SGLDUNETLoader` 类概览
```python
class SGLDUNETLoader:
```
**EN:** This block defines class `SGLDUNETLoader`. It encapsulates sgldunetloader behavior.
**CN:** 该代码块定义了类 `SGLDUNETLoader`。 它用于封装 sgldunetloader 相关行为。

### Lines 167-168: `__init__` implementation / `__init__` 实现
```python
    def __init__(self):
        self.generator = SGLDiffusionGenerator()
```
**EN:** This block defines method `__init__` on `SGLDUNETLoader`. It initializes the instance state. Key calls include `SGLDiffusionGenerator`.
**CN:** 该代码块定义了 `SGLDUNETLoader` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `SGLDiffusionGenerator`。

### Lines 170-180: `INPUT_TYPES` implementation / `INPUT_TYPES` 实现
```python
    @classmethod
    def INPUT_TYPES(s):
        return {
            "required": {
                "unet_name": (folder_paths.get_filename_list("diffusion_models"),),
                "weight_dtype": (["default", "fp8_e4m3fn", "fp8_e5m2"],),
            },
            "optional": {
                "sgld_options": ("SGLD_OPTIONS",),
            },
        }
```
**EN:** This block defines method `INPUT_TYPES` on `SGLDUNETLoader`. It handles input types logic. Key calls include `folder_paths.get_filename_list`. Parameters such as `s` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDUNETLoader` 的方法 `INPUT_TYPES`。 它用于处理 input types 相关逻辑。 关键调用包括 `folder_paths.get_filename_list`。 本段逻辑主要由 `s` 等参数驱动。

### Lines 182-185: supporting statements / 辅助语句
```python
    RETURN_TYPES = ("MODEL",)
    FUNCTION = "load_unet"

    CATEGORY = "SGLDiffusion"
```
**EN:** This block gathers supporting statements inside `SGLDUNETLoader`. It updates names such as `RETURN_TYPES`, `FUNCTION`, and `CATEGORY`.
**CN:** 该代码块汇集了位于 `SGLDUNETLoader` 内部的辅助语句。 它会更新 `RETURN_TYPES`、`FUNCTION` 和 `CATEGORY` 等名称。

### Lines 187-199: `load_unet` implementation / `load_unet` 实现
```python
    def load_unet(self, unet_name, weight_dtype, sgld_options: dict = None):
        model_options = {}
        if weight_dtype == "fp8_e4m3fn":
            model_options["dtype"] = torch.float8_e4m3fn
        elif weight_dtype == "fp8_e5m2":
            model_options["dtype"] = torch.float8_e5m2

        unet_path = folder_paths.get_full_path("diffusion_models", unet_name)

        model = self.generator.load_model(
            unet_path, model_options=model_options, sgld_options=sgld_options
        )
        return (model,)
```
**EN:** This block defines method `load_unet` on `SGLDUNETLoader`. It loads unet. Key calls include `folder_paths.get_full_path`, and `self.generator.load_model`. The implementation branches on conditions. Parameters such as `unet_name`, `weight_dtype`, and `sgld_options` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDUNETLoader` 的方法 `load_unet`。 它用于加载unet。 关键调用包括 `folder_paths.get_full_path` 和 `self.generator.load_model`。 实现中包含条件分支。 本段逻辑主要由 `unet_name`、`weight_dtype` 和 `sgld_options` 等参数驱动。

### Lines 202-204: `SGLDiffusionServerModel` class overview / `SGLDiffusionServerModel` 类概览
```python
class SGLDiffusionServerModel:
    """Node to load and manage SGLang Diffusion server connection."""
```
**EN:** This block defines class `SGLDiffusionServerModel`. Node to load and manage SGLang Diffusion server connection.
**CN:** 该代码块定义了类 `SGLDiffusionServerModel`。 它用于封装 sgldiffusion server model 相关行为。

### Lines 205-224: `INPUT_TYPES` implementation / `INPUT_TYPES` 实现
```python
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "base_url": (
                    "STRING",
                    {
                        "default": "http://localhost:3000/v1",
                        "multiline": False,
                    },
                ),
                "api_key": (
                    "STRING",
                    {
                        "default": "sk-proj-1234567890",
                        "multiline": False,
                    },
                ),
            }
        }
```
**EN:** This block defines method `INPUT_TYPES` on `SGLDiffusionServerModel`. It handles input types logic.
**CN:** 该代码块定义了 `SGLDiffusionServerModel` 的方法 `INPUT_TYPES`。 它用于处理 input types 相关逻辑。

### Lines 226-229: supporting statements / 辅助语句
```python
    RETURN_TYPES = ("SGLD_CLIENT", "STRING")
    RETURN_NAMES = ("sgld_client", "model_info")
    FUNCTION = "load_server"
    CATEGORY = "SGLDiffusion"
```
**EN:** This block gathers supporting statements inside `SGLDiffusionServerModel`. It updates names such as `RETURN_TYPES`, `RETURN_NAMES`, `FUNCTION`, and `CATEGORY`.
**CN:** 该代码块汇集了位于 `SGLDiffusionServerModel` 内部的辅助语句。 它会更新 `RETURN_TYPES`、`RETURN_NAMES`、`FUNCTION` 和 `CATEGORY` 等名称。

### Lines 231-243: `load_server` implementation / `load_server` 实现
```python
    def load_server(self, base_url: str, api_key: str):
        """Initialize OpenAI client for SGLang Diffusion server."""
        client = SGLDiffusionServerAPI(base_url=base_url, api_key=api_key)
        try:
            model_info = client.get_model_info()
            # Format model_info as a readable string
            info_lines = ["=== SGLDiffusion Model Info ==="]
            for key, value in model_info.items():
                info_lines.append(f"{key}: {value}")
            model_info_str = "\n".join(info_lines)
        except Exception as e:
            model_info_str = f"Failed to get model info: {str(e)}"
        return (client, model_info_str)
```
**EN:** This block defines method `load_server` on `SGLDiffusionServerModel`. Initialize OpenAI client for SGLang Diffusion server. Key calls include `SGLDiffusionServerAPI`, `client.get_model_info`, `model_info.items`, `join`, and `info_lines.append`. The implementation iterates over collections or steps, handles exceptional paths. Parameters such as `base_url`, and `api_key` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionServerModel` 的方法 `load_server`。 它用于加载server。 关键调用包括 `SGLDiffusionServerAPI`、`client.get_model_info`、`model_info.items`、`join` 和 `info_lines.append`。 实现中会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `base_url` 和 `api_key` 等参数驱动。

### Lines 246-248: `SGLDiffusionGenerateImage` class overview / `SGLDiffusionGenerateImage` 类概览
```python
class SGLDiffusionGenerateImage:
    """Node to generate images using SGLang Diffusion."""
```
**EN:** This block defines class `SGLDiffusionGenerateImage`. Node to generate images using SGLang Diffusion.
**CN:** 该代码块定义了类 `SGLDiffusionGenerateImage`。 它用于封装 sgldiffusion generate image 相关行为。

### Lines 249-328: `INPUT_TYPES` implementation / `INPUT_TYPES` 实现
```python
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "sgld_client": ("SGLD_CLIENT",),
                "positive_prompt": (
                    "STRING",
                    {
                        "default": "",
                        "tooltip": "Text prompt for image generation",
                    },
                ),
            },
            "optional": {
                "negative_prompt": (
                    "STRING",
                    {
                        "default": "",
                        "tooltip": "Negative prompt to avoid certain elements",
                    },
                ),
                "image": (
                    "IMAGE",
                    {
                        "default": None,
                        "tooltip": "input image to use for editing",
                    },
                ),
                "seed": (
                    "INT",
                    {
                        "default": 1024,
                        "min": -1,
                        "max": 2**32 - 1,
                    },
                ),
                "steps": (
                    "INT",
                    {
                        "default": 6,
                        "min": 1,
                        "max": 100,
                        "step": 1,
                    },
                ),
                "cfg": (
                    "FLOAT",
                    {
                        "default": 7.0,
                        "min": 1.0,
                        "max": 20.0,
                        "step": 0.1,
                    },
                ),
                "width": (
                    "INT",
                    {
                        "default": 1024,
                        "min": 256,
                        "max": 4096,
                        "step": 64,
                    },
                ),
                "height": (
                    "INT",
                    {
                        "default": 1024,
                        "min": 256,
                        "max": 4096,
                        "step": 64,
                    },
                ),
                "enable_teacache": (
                    "BOOLEAN",
                    {
                        "default": False,
                    },
                ),
            },
        }
```
**EN:** This block defines method `INPUT_TYPES` on `SGLDiffusionGenerateImage`. It handles input types logic.
**CN:** 该代码块定义了 `SGLDiffusionGenerateImage` 的方法 `INPUT_TYPES`。 它用于处理 input types 相关逻辑。

### Lines 330-334: supporting statements / 辅助语句
```python
    RETURN_TYPES = ("IMAGE",)
    RETURN_NAMES = ("image",)
    FUNCTION = "generate_image"
    CATEGORY = "SGLDiffusion"
    OUTPUT_NODE = False
```
**EN:** This block gathers supporting statements inside `SGLDiffusionGenerateImage`. It updates names such as `RETURN_TYPES`, `RETURN_NAMES`, `FUNCTION`, `CATEGORY`, and `OUTPUT_NODE`.
**CN:** 该代码块汇集了位于 `SGLDiffusionGenerateImage` 内部的辅助语句。 它会更新 `RETURN_TYPES`、`RETURN_NAMES`、`FUNCTION`、`CATEGORY` 和 `OUTPUT_NODE` 等名称。

### Lines 336-394: `generate_image` implementation / `generate_image` 实现
```python
    def generate_image(
        self,
        sgld_client: SGLDiffusionServerAPI,
        positive_prompt: str,
        negative_prompt: str = "",
        image: torch.Tensor = None,
        seed: int = 1024,
        steps: int = 6,
        cfg: float = 7.0,
        width: int = 1024,
        height: int = 1024,
        enable_teacache: bool = False,
    ):
        """Generate image using SGLang Diffusion API."""
        if not positive_prompt:
            raise ValueError("Prompt cannot be empty")

        size = f"{width}x{height}"

        # Prepare request parameters
        request_params = {
            "prompt": positive_prompt,
            "size": size,
            "response_format": "b64_json",
        }

        # Add optional parameters if provided
        if negative_prompt:
            request_params["negative_prompt"] = negative_prompt
        if cfg is not None:
            request_params["guidance_scale"] = cfg
        if steps is not None:
            request_params["num_inference_steps"] = steps
        if seed is not None and seed >= 0:
            request_params["seed"] = seed
        if enable_teacache:
            request_params["enable_teacache"] = True
        if image is not None:
            # If the image is empty, use the size of the image to generate the image
            if is_empty_image(image):
                width, height = image.shape[2], image.shape[1]
                size = f"{width}x{height}"
                request_params["size"] = size
            else:
                request_params["image_path"] = get_image_path(image)

        # Call API
        try:
            response = sgld_client.generate_image(**request_params)
        except Exception as e:
            raise RuntimeError(f"Failed to generate image: {str(e)}")

        # Decode base64 image
        if not response["data"] or not response["data"][0]["b64_json"]:
            raise RuntimeError("No image data in response")
        image_data = response["data"][0]["b64_json"]
        image = convert_b64_to_tensor_image(image_data)

        return (image,)
```
**EN:** This block defines method `generate_image` on `SGLDiffusionGenerateImage`. Generate image using SGLang Diffusion API. Key calls include `convert_b64_to_tensor_image`, `ValueError`, `is_empty_image`, `sgld_client.generate_image`, and `RuntimeError`. The implementation branches on conditions, handles exceptional paths. Parameters such as `sgld_client`, `positive_prompt`, `negative_prompt`, `image`, and `seed` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionGenerateImage` 的方法 `generate_image`。 它用于生成image。 关键调用包括 `convert_b64_to_tensor_image`、`ValueError`、`is_empty_image`、`sgld_client.generate_image` 和 `RuntimeError`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `sgld_client`、`positive_prompt`、`negative_prompt`、`image` 和 `seed` 等参数驱动。

### Lines 397-399: `SGLDiffusionGenerateVideo` class overview / `SGLDiffusionGenerateVideo` 类概览
```python
class SGLDiffusionGenerateVideo:
    """Node to generate videos using SGLang Diffusion."""
```
**EN:** This block defines class `SGLDiffusionGenerateVideo`. Node to generate videos using SGLang Diffusion.
**CN:** 该代码块定义了类 `SGLDiffusionGenerateVideo`。 它用于封装 sgldiffusion generate video 相关行为。

### Lines 400-506: `INPUT_TYPES` implementation / `INPUT_TYPES` 实现
```python
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "sgld_client": ("SGLD_CLIENT",),
                "positive_prompt": (
                    "STRING",
                    {
                        "default": "",
                        "tooltip": "Text prompt for video generation",
                    },
                ),
            },
            "optional": {
                "negative_prompt": (
                    "STRING",
                    {
                        "default": "",
                        "tooltip": "Negative prompt to avoid certain elements",
                    },
                ),
                "image": (
                    "IMAGE",
                    {
                        "default": None,
                        "tooltip": "input image to use for image-to-video",
                    },
                ),
                "seed": (
                    "INT",
                    {
                        "default": 1024,
                        "min": -1,
                        "max": 2**32 - 1,
                    },
                ),
                "steps": (
                    "INT",
                    {
                        "default": 6,
                        "min": 1,
                        "max": 100,
                        "step": 1,
                    },
                ),
                "cfg": (
                    "FLOAT",
                    {
                        "default": 7.0,
                        "min": 1.0,
                        "max": 20.0,
                        "step": 0.1,
                    },
                ),
                "width": (
                    "INT",
                    {
                        "default": 1280,
                        "min": 256,
                        "max": 4096,
                        "step": 1,
                    },
                ),
                "height": (
                    "INT",
                    {
                        "default": 720,
                        "min": 256,
                        "max": 4096,
                        "step": 1,
                    },
                ),
                "num_frames": (
                    "INT",
                    {
                        "default": 120,
                        "min": 1,
                        "max": 1000,
                        "step": 1,
                    },
                ),
                "fps": (
                    "INT",
                    {
                        "default": 24,
                        "min": 1,
                        "max": 60,
                        "step": 1,
                    },
                ),
                "seconds": (
                    "INT",
                    {
                        "default": 5,
                        "min": 1,
                        "max": 60,
                        "step": 1,
                    },
                ),
                "enable_teacache": (
                    "BOOLEAN",
                    {
                        "default": False,
                    },
                ),
            },
        }
```
**EN:** This block defines method `INPUT_TYPES` on `SGLDiffusionGenerateVideo`. It handles input types logic.
**CN:** 该代码块定义了 `SGLDiffusionGenerateVideo` 的方法 `INPUT_TYPES`。 它用于处理 input types 相关逻辑。

### Lines 508-512: supporting statements / 辅助语句
```python
    RETURN_TYPES = ("VIDEO", "STRING")
    RETURN_NAMES = ("video", "video_path")
    FUNCTION = "generate_video"
    CATEGORY = "SGLDiffusion"
    OUTPUT_NODE = False
```
**EN:** This block gathers supporting statements inside `SGLDiffusionGenerateVideo`. It updates names such as `RETURN_TYPES`, `RETURN_NAMES`, `FUNCTION`, `CATEGORY`, and `OUTPUT_NODE`.
**CN:** 该代码块汇集了位于 `SGLDiffusionGenerateVideo` 内部的辅助语句。 它会更新 `RETURN_TYPES`、`RETURN_NAMES`、`FUNCTION`、`CATEGORY` 和 `OUTPUT_NODE` 等名称。

### Lines 514-576: `generate_video` implementation / `generate_video` 实现
```python
    def generate_video(
        self,
        sgld_client: SGLDiffusionServerAPI,
        positive_prompt: str,
        negative_prompt: str = "",
        image: torch.Tensor = None,
        seed: int = 1024,
        steps: int = 6,
        cfg: float = 7.0,
        width: int = 1280,
        height: int = 720,
        num_frames: int = 120,
        fps: int = 24,
        seconds: int = 5,
        enable_teacache: bool = False,
    ):
        """Generate video using SGLang Diffusion API."""
        if not positive_prompt:
            raise ValueError("Prompt cannot be empty")

        size = f"{width}x{height}"
        output_dir = folder_paths.get_temp_directory()

        # Prepare request parameters
        request_params = {
            "prompt": positive_prompt,
            "size": size,
            "seconds": seconds,
            "fps": fps,
            "output_path": output_dir,
        }

        # Add optional parameters if provided
        if negative_prompt:
            request_params["negative_prompt"] = negative_prompt
        if cfg is not None:
            request_params["guidance_scale"] = cfg
        if steps is not None:
            request_params["num_inference_steps"] = steps
        if seed is not None and seed >= 0:
            request_params["seed"] = seed
        if enable_teacache:
            request_params["enable_teacache"] = True
        if num_frames is not None:
            request_params["num_frames"] = num_frames
        if image is not None:
            # If the image is empty, use the size of the image to generate the video
            if is_empty_image(image):
                width, height = image.shape[2], image.shape[1]
                size = f"{width}x{height}"
                request_params["size"] = size
            else:
                request_params["input_reference"] = get_image_path(image)

        # Call API
        try:
            response = sgld_client.generate_video(**request_params)
            video_path = response.get("file_path", "")
            video = convert_video_to_comfy_video(video_path, height, width)
        except Exception as e:
            raise RuntimeError(f"Failed to generate video: {str(e)}")

        return (video, video_path)
```
**EN:** This block defines method `generate_video` on `SGLDiffusionGenerateVideo`. Generate video using SGLang Diffusion API. Key calls include `folder_paths.get_temp_directory`, `ValueError`, `is_empty_image`, `sgld_client.generate_video`, and `response.get`. The implementation branches on conditions, handles exceptional paths. Parameters such as `sgld_client`, `positive_prompt`, `negative_prompt`, `image`, and `seed` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionGenerateVideo` 的方法 `generate_video`。 它用于生成video。 关键调用包括 `folder_paths.get_temp_directory`、`ValueError`、`is_empty_image`、`sgld_client.generate_video` 和 `response.get`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `sgld_client`、`positive_prompt`、`negative_prompt`、`image` 和 `seed` 等参数驱动。

### Lines 579-581: `SGLDiffusionServerSetLora` class overview / `SGLDiffusionServerSetLora` 类概览
```python
class SGLDiffusionServerSetLora:
    """Node to set LoRA adapter for SGLang Diffusion server."""
```
**EN:** This block defines class `SGLDiffusionServerSetLora`. Node to set LoRA adapter for SGLang Diffusion server.
**CN:** 该代码块定义了类 `SGLDiffusionServerSetLora`。 它用于封装 sgldiffusion server set lora 相关行为。

### Lines 582-616: `INPUT_TYPES` implementation / `INPUT_TYPES` 实现
```python
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "sgld_client": ("SGLD_CLIENT",),
                "lora_name": (
                    "STRING",
                    {
                        "default": "",
                        "tooltip": "The name of the LoRA adapter",
                    },
                ),
            },
            "optional": {
                "lora_nickname": (
                    "STRING",
                    {
                        "default": "",
                        "tooltip": "The nickname of the LoRA adapter",
                    },
                ),
                "target": (
                    [
                        "all",
                        "transformer",
                        "transformer_2",
                        "critic",
                    ],
                    {
                        "default": "all",
                        "tooltip": "Which transformer(s) to apply the LoRA to",
                    },
                ),
            },
        }
```
**EN:** This block defines method `INPUT_TYPES` on `SGLDiffusionServerSetLora`. It handles input types logic.
**CN:** 该代码块定义了 `SGLDiffusionServerSetLora` 的方法 `INPUT_TYPES`。 它用于处理 input types 相关逻辑。

### Lines 618-622: supporting statements / 辅助语句
```python
    RETURN_TYPES = ("SGLD_CLIENT",)
    RETURN_NAMES = ("sgld_client",)
    FUNCTION = "set_lora"
    CATEGORY = "SGLDiffusion"
    OUTPUT_NODE = False
```
**EN:** This block gathers supporting statements inside `SGLDiffusionServerSetLora`. It updates names such as `RETURN_TYPES`, `RETURN_NAMES`, `FUNCTION`, `CATEGORY`, and `OUTPUT_NODE`.
**CN:** 该代码块汇集了位于 `SGLDiffusionServerSetLora` 内部的辅助语句。 它会更新 `RETURN_TYPES`、`RETURN_NAMES`、`FUNCTION`、`CATEGORY` 和 `OUTPUT_NODE` 等名称。

### Lines 624-647: `set_lora` implementation / `set_lora` 实现
```python
    def set_lora(
        self,
        sgld_client: SGLDiffusionServerAPI,
        lora_name: str = "",
        lora_nickname: str = "",
        target: str = "all",
    ):
        """Set LoRA adapter using SGLang Diffusion API."""
        if lora_nickname == "":
            lora_nickname = os.path.splitext(lora_name)[0]

        # Prepare request parameters
        request_params = {
            "lora_nickname": lora_nickname,
            "lora_path": lora_name,
            "target": target,
        }

        # Call API
        try:
            response = sgld_client.set_lora(**request_params)
            return (sgld_client,)
        except Exception as e:
            raise RuntimeError(f"Failed to set LoRA adapter: {str(e)}")
```
**EN:** This block defines method `set_lora` on `SGLDiffusionServerSetLora`. Set LoRA adapter using SGLang Diffusion API. Key calls include `sgld_client.set_lora`, `os.path.splitext`, `RuntimeError`, and `str`. The implementation branches on conditions, handles exceptional paths. Parameters such as `sgld_client`, `lora_name`, `lora_nickname`, and `target` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionServerSetLora` 的方法 `set_lora`。 它用于设置lora。 关键调用包括 `sgld_client.set_lora`、`os.path.splitext`、`RuntimeError` 和 `str`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `sgld_client`、`lora_name`、`lora_nickname` 和 `target` 等参数驱动。

### Lines 650-652: `SGLDiffusionServerUnsetLora` class overview / `SGLDiffusionServerUnsetLora` 类概览
```python
class SGLDiffusionServerUnsetLora:
    """Node to unset LoRA adapter for SGLang Diffusion server."""
```
**EN:** This block defines class `SGLDiffusionServerUnsetLora`. Node to unset LoRA adapter for SGLang Diffusion server.
**CN:** 该代码块定义了类 `SGLDiffusionServerUnsetLora`。 它用于封装 sgldiffusion server unset lora 相关行为。

### Lines 653-673: `INPUT_TYPES` implementation / `INPUT_TYPES` 实现
```python
    @classmethod
    def INPUT_TYPES(cls):
        return {
            "required": {
                "sgld_client": ("SGLD_CLIENT",),
            },
            "optional": {
                "target": (
                    [
                        "all",
                        "transformer",
                        "transformer_2",
                        "critic",
                    ],
                    {
                        "default": "all",
                        "tooltip": "Which transformer(s) to unset the LoRA from",
                    },
                ),
            },
        }
```
**EN:** This block defines method `INPUT_TYPES` on `SGLDiffusionServerUnsetLora`. It handles input types logic.
**CN:** 该代码块定义了 `SGLDiffusionServerUnsetLora` 的方法 `INPUT_TYPES`。 它用于处理 input types 相关逻辑。

### Lines 675-679: supporting statements / 辅助语句
```python
    RETURN_TYPES = ("SGLD_CLIENT",)
    RETURN_NAMES = ("sgld_client",)
    FUNCTION = "unset_lora"
    CATEGORY = "SGLDiffusion"
    OUTPUT_NODE = False
```
**EN:** This block gathers supporting statements inside `SGLDiffusionServerUnsetLora`. It updates names such as `RETURN_TYPES`, `RETURN_NAMES`, `FUNCTION`, `CATEGORY`, and `OUTPUT_NODE`.
**CN:** 该代码块汇集了位于 `SGLDiffusionServerUnsetLora` 内部的辅助语句。 它会更新 `RETURN_TYPES`、`RETURN_NAMES`、`FUNCTION`、`CATEGORY` 和 `OUTPUT_NODE` 等名称。

### Lines 681-691: `unset_lora` implementation / `unset_lora` 实现
```python
    def unset_lora(
        self,
        sgld_client: SGLDiffusionServerAPI,
        target: str = "all",
    ):
        """Unset LoRA adapter using SGLang Diffusion API."""
        try:
            response = sgld_client.unset_lora(target=target)
            return (sgld_client,)
        except Exception as e:
            raise RuntimeError(f"Failed to unset LoRA adapter: {str(e)}")
```
**EN:** This block defines method `unset_lora` on `SGLDiffusionServerUnsetLora`. Unset LoRA adapter using SGLang Diffusion API. Key calls include `sgld_client.unset_lora`, `RuntimeError`, and `str`. The implementation handles exceptional paths. Parameters such as `sgld_client`, and `target` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDiffusionServerUnsetLora` 的方法 `unset_lora`。 它用于处理 unset lora 相关逻辑。 关键调用包括 `sgld_client.unset_lora`、`RuntimeError` 和 `str`。 实现中处理异常路径。 本段逻辑主要由 `sgld_client` 和 `target` 等参数驱动。

### Lines 695-715: supporting statements / 辅助语句
```python
NODE_CLASS_MAPPINGS = {
    "SGLDiffusionServerModel": SGLDiffusionServerModel,
    "SGLDiffusionGenerateImage": SGLDiffusionGenerateImage,
    "SGLDiffusionGenerateVideo": SGLDiffusionGenerateVideo,
    "SGLDiffusionServerSetLora": SGLDiffusionServerSetLora,
    "SGLDiffusionServerUnsetLora": SGLDiffusionServerUnsetLora,
    "SGLDUNETLoader": SGLDUNETLoader,
    "SGLDOptions": SGLDOptions,
    "SGLDLoraLoader": SGLDLoraLoader,
}

NODE_DISPLAY_NAME_MAPPINGS = {
    "SGLDiffusionServerModel": "SGLDiffusion Server Model",
    "SGLDiffusionGenerateImage": "SGLDiffusion Generate Image",
    "SGLDiffusionGenerateVideo": "SGLDiffusion Generate Video",
    "SGLDiffusionServerSetLora": "SGLDiffusion Server Set LoRA",
    "SGLDiffusionServerUnsetLora": "SGLDiffusion Server Unset LoRA",
    "SGLDUNETLoader": "SGLDiffusion UNET Loader",
    "SGLDOptions": "SGLDiffusion Options",
    "SGLDLoraLoader": "SGLDiffusion LoRA Loader",
}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `NODE_CLASS_MAPPINGS`, and `NODE_DISPLAY_NAME_MAPPINGS`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `NODE_CLASS_MAPPINGS` 和 `NODE_DISPLAY_NAME_MAPPINGS` 等名称。

## Key Concepts / 关键概念
- `SGLDOptions`: Primary class that encapsulates sgldoptions behavior. / 核心类，用于封装 sgldoptions 相关行为。
- `SGLDLoraLoader`: Primary class that encapsulates sgldlora loader behavior. / 核心类，用于封装 sgldlora loader 相关行为。
- `SGLDUNETLoader`: Primary class that encapsulates sgldunetloader behavior. / 核心类，用于封装 sgldunetloader 相关行为。
- `SGLDiffusionServerModel`: Node to load and manage SGLang Diffusion server connection. / 核心类，用于封装 sgldiffusion server model 相关行为。
- `SGLDiffusionGenerateImage`: Node to generate images using SGLang Diffusion. / 核心类，用于封装 sgldiffusion generate image 相关行为。
- `SGLDiffusionGenerateVideo`: Node to generate videos using SGLang Diffusion. / 核心类，用于封装 sgldiffusion generate video 相关行为。
- `SGLDiffusionServerSetLora`: Node to set LoRA adapter for SGLang Diffusion server. / 核心类，用于封装 sgldiffusion server set lora 相关行为。
- `SGLDiffusionServerUnsetLora`: Node to unset LoRA adapter for SGLang Diffusion server. / 核心类，用于封装 sgldiffusion server unset lora 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `uuid`
- **Third-party / 第三方依赖**: `folder_paths`, `torch`
- **Internal modules / 内部模块**: `.core`, `.utils`

- **Total lines / 总行数**: 715

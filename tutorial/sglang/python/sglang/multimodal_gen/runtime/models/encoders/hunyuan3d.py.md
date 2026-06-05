# hunyuan3d.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/encoders/hunyuan3d.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for hunyuan3d within the multimodal runtime. Key symbols include `get_1d_sincos_pos_embed_from_grid`, `ImageEncoder`, `CLIPImageEncoder`. / 该模块实现多模态运行时中与 hunyuan3d 相关的模型构件。 关键符号包括 `get_1d_sincos_pos_embed_from_grid`, `ImageEncoder`, `CLIPImageEncoder`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/Tencent-Hunyuan/Hunyuan3D-2

import numpy as np
import torch
import torch.nn as nn
from torchvision import transforms
from transformers import (
    CLIPVisionConfig,
    CLIPVisionModelWithProjection,
    Dinov2Config,
    Dinov2Model,
)

from sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload import (
    LayerwiseOffloadableModuleMixin,
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 19-32: Function `get_1d_sincos_pos_embed_from_grid` / 函数 `get_1d_sincos_pos_embed_from_grid`
```python
def get_1d_sincos_pos_embed_from_grid(embed_dim, pos):

    assert embed_dim % 2 == 0
    omega = np.arange(embed_dim // 2, dtype=np.float64)
    omega /= embed_dim / 2.0
    omega = 1.0 / 10000**omega  # (D/2,)

    pos = pos.reshape(-1)  # (M,)
    out = np.einsum("m,d->md", pos, omega)  # (M, D/2), outer product

    emb_sin = np.sin(out)  # (M, D/2)
    emb_cos = np.cos(out)  # (M, D/2)

    return np.concatenate([emb_sin, emb_cos], axis=1)
```
**EN:** This function drives `get_1d_sincos_pos_embed_from_grid` with inputs such as `embed_dim`, `pos`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_1d_sincos_pos_embed_from_grid`，主要处理 `embed_dim`, `pos` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 35-107: Class `ImageEncoder` / 类 `ImageEncoder`
```python
class ImageEncoder(nn.Module, LayerwiseOffloadableModuleMixin):
    layerwise_offload_dit_group_enabled = False
    layer_names = [
        "model.encoder.layer",
        "model.vision_model.encoder.layers",
    ]
    MODEL_CLASS = None
    MODEL_CONFIG_CLASS = None
    mean = []
    std = []

    def __init__(
        self,
        version=None,
# ...
            dtype=dtype,
        )

        return zero
```
**EN:** This class models `ImageEncoder` as a specialization of `nn.Module`, `LayerwiseOffloadableModuleMixin`. Important methods include `__init__`, `forward`, `unconditional_embedding`.
**CN:** 该类实现 `ImageEncoder`，并继承/扩展 `nn.Module`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `__init__`, `forward`, `unconditional_embedding`。

### Lines 110-114: Class `CLIPImageEncoder` / 类 `CLIPImageEncoder`
```python
class CLIPImageEncoder(ImageEncoder):
    MODEL_CLASS = CLIPVisionModelWithProjection
    MODEL_CONFIG_CLASS = CLIPVisionConfig
    mean = [0.48145466, 0.4578275, 0.40821073]
    std = [0.26862954, 0.26130258, 0.27577711]
```
**EN:** This class models `CLIPImageEncoder` as a specialization of `ImageEncoder`.
**CN:** 该类实现 `CLIPImageEncoder`，并继承/扩展 `ImageEncoder`。

### Lines 117-121: Class `DinoImageEncoder` / 类 `DinoImageEncoder`
```python
class DinoImageEncoder(ImageEncoder):
    MODEL_CLASS = Dinov2Model
    MODEL_CONFIG_CLASS = Dinov2Config
    mean = [0.485, 0.456, 0.406]
    std = [0.229, 0.224, 0.225]
```
**EN:** This class models `DinoImageEncoder` as a specialization of `ImageEncoder`.
**CN:** 该类实现 `DinoImageEncoder`，并继承/扩展 `ImageEncoder`。

### Lines 124-201: Class `DinoImageEncoderMV` / 类 `DinoImageEncoderMV`
```python
class DinoImageEncoderMV(DinoImageEncoder):
    _aliases = [
        "hy3dshape.models.conditioner.DinoImageEncoderMV",
    ]

    def __init__(
        self,
        version=None,
        config=None,
        use_cls_token=True,
        image_size=224,
        view_num=4,
        **kwargs,
    ):
# ...
            device=device,
            dtype=dtype,
        )
        return zero
```
**EN:** This class models `DinoImageEncoderMV` as a specialization of `DinoImageEncoder`. Important methods include `__init__`, `forward`, `unconditional_embedding`.
**CN:** 该类实现 `DinoImageEncoderMV`，并继承/扩展 `DinoImageEncoder`。 其中较重要的方法包括 `__init__`, `forward`, `unconditional_embedding`。

### Lines 204-212: Function `build_image_encoder` / 函数 `build_image_encoder`
```python
def build_image_encoder(config):
    if config["type"] == "CLIPImageEncoder":
        return CLIPImageEncoder(**config["kwargs"])
    elif config["type"] == "DinoImageEncoder":
        return DinoImageEncoder(**config["kwargs"])
    elif config["type"] == "DinoImageEncoderMV":
        return DinoImageEncoderMV(**config["kwargs"])
    else:
        raise ValueError(f'Unknown image encoder type: {config["type"]}')
```
**EN:** This function drives `build_image_encoder` with inputs such as `config`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `build_image_encoder`，主要处理 `config` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 215-249: Class `DualImageEncoder` / 类 `DualImageEncoder`
```python
class DualImageEncoder(nn.Module, LayerwiseOffloadableModuleMixin):
    layerwise_offload_dit_group_enabled = False
    layer_names = [
        "main_image_encoder.model.encoder.layer",
        "main_image_encoder.model.vision_model.encoder.layers",
        "additional_image_encoder.model.encoder.layer",
        "additional_image_encoder.model.vision_model.encoder.layers",
    ]

    def __init__(
        self,
        main_image_encoder,
        additional_image_encoder,
    ):
# ...
                batch_size, **kwargs
            ),
        }
        return outputs
```
**EN:** This class models `DualImageEncoder` as a specialization of `nn.Module`, `LayerwiseOffloadableModuleMixin`. Important methods include `__init__`, `forward`, `unconditional_embedding`.
**CN:** 该类实现 `DualImageEncoder`，并继承/扩展 `nn.Module`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `__init__`, `forward`, `unconditional_embedding`。

### Lines 252-278: Class `SingleImageEncoder` / 类 `SingleImageEncoder`
```python
class SingleImageEncoder(nn.Module, LayerwiseOffloadableModuleMixin):
    layerwise_offload_dit_group_enabled = False
    layer_names = [
        "main_image_encoder.model.encoder.layer",
        "main_image_encoder.model.vision_model.encoder.layers",
    ]

    def __init__(
        self,
        main_image_encoder,
    ):
        super().__init__()
        self.main_image_encoder = build_image_encoder(main_image_encoder)

# ...
                batch_size, **kwargs
            ),
        }
        return outputs
```
**EN:** This class models `SingleImageEncoder` as a specialization of `nn.Module`, `LayerwiseOffloadableModuleMixin`. Important methods include `__init__`, `forward`, `unconditional_embedding`.
**CN:** 该类实现 `SingleImageEncoder`，并继承/扩展 `nn.Module`, `LayerwiseOffloadableModuleMixin`。 其中较重要的方法包括 `__init__`, `forward`, `unconditional_embedding`。

### Lines 279-287: Top-level configuration / 顶层配置
```python


# Entry class for model registry
EntryClass = [
    SingleImageEncoder,
    DualImageEncoder,
    DinoImageEncoder,
    DinoImageEncoderMV,
]
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Command-line interface / 命令行接口
- Image generation flow / 图像生成流程
- Symbol `get_1d_sincos_pos_embed_from_grid` anchors the module API / 符号 `get_1d_sincos_pos_embed_from_grid` 构成该模块的核心 API
- Symbol `ImageEncoder` anchors the module API / 符号 `ImageEncoder` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`
- **External / 外部**: `numpy`, `torch`, `torch.nn`, `torchvision`, `transformers`

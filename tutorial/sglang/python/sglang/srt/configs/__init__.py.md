# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Exports configuration helpers and model-specific config modules used by the SGLang runtime. / 导出 SGLang 运行时使用的配置辅助函数与模型专用配置模块。

## Line-by-Line Analysis / 逐行分析

### Lines 1-39: Imports dependencies
```python
from sglang.srt.configs.afmoe import AfmoeConfig
from sglang.srt.configs.bailing_hybrid import BailingHybridConfig
from sglang.srt.configs.chatglm import ChatGLMConfig
from sglang.srt.configs.dbrx import DbrxConfig
from sglang.srt.configs.deepseekvl2 import DeepseekVL2Config
from sglang.srt.configs.dots_ocr import DotsOCRConfig
from sglang.srt.configs.dots_vlm import DotsVLMConfig
from sglang.srt.configs.exaone import ExaoneConfig
from sglang.srt.configs.falcon_h1 import FalconH1Config
from sglang.srt.configs.granitemoehybrid import GraniteMoeHybridConfig
from sglang.srt.configs.interns2preview import InternS2PreviewConfig
from sglang.srt.configs.janus_pro import MultiModalityConfig
from sglang.srt.configs.jet_nemotron import JetNemotronConfig
from sglang.srt.configs.jet_vlm import JetVLMConfig
from sglang.srt.configs.kimi_k25 import KimiK25Config
from sglang.srt.configs.kimi_linear import KimiLinearConfig
from sglang.srt.configs.kimi_vl import KimiVLConfig
from sglang.srt.configs.kimi_vl_moonvit import MoonViTConfig
from sglang.srt.configs.laguna import LagunaConfig
from sglang.srt.configs.lfm2 import Lfm2Config
from sglang.srt.configs.lfm2_moe import Lfm2MoeConfig
from sglang.srt.configs.lfm2_vl import Lfm2VlConfig
from sglang.srt.configs.longcat_flash import LongcatFlashConfig
from sglang.srt.configs.minicpmv4_6 import MiniCPMV4_6Config, MiniCPMV4_6VisionConfig
from sglang.srt.configs.nano_nemotron_vl import (
    NemotronH_Nano_Omni_Reasoning_V3_Config,
    NemotronH_Nano_VL_V2_Config,
)
from sglang.srt.configs.nemotron_h import NemotronHConfig
from sglang.srt.configs.olmo3 import Olmo3Config
from sglang.srt.configs.qwen3_5 import Qwen3_5Config, Qwen3_5MoeConfig
from sglang.srt.configs.qwen3_asr import Qwen3ASRConfig
from sglang.srt.configs.qwen3_next import Qwen3NextConfig
from sglang.srt.configs.step3_vl import (
    Step3TextConfig,
    Step3VisionEncoderConfig,
    Step3VLConfig,
)
from sglang.srt.configs.step3p5 import Step3p5Config
```
**EN:** This block groups related imports for the module, including sglang.srt.configs.afmoe.AfmoeConfig, sglang.srt.configs.bailing_hybrid.BailingHybridConfig, sglang.srt.configs.chatglm.ChatGLMConfig, sglang.srt.configs.dbrx.DbrxConfig, sglang.srt.configs.deepseekvl2.DeepseekVL2Config, and 32 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 sglang.srt.configs.afmoe.AfmoeConfig, sglang.srt.configs.bailing_hybrid.BailingHybridConfig, sglang.srt.configs.chatglm.ChatGLMConfig, sglang.srt.configs.dbrx.DbrxConfig, sglang.srt.configs.deepseekvl2.DeepseekVL2Config 等 32 项，为后续代码准备所需名称。

### Lines 40-40: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 41-79: Declares __all__
```python
__all__ = [
    "AfmoeConfig",
    "BailingHybridConfig",
    "ExaoneConfig",
    "ChatGLMConfig",
    "DbrxConfig",
    "DeepseekVL2Config",
    "LongcatFlashConfig",
    "MultiModalityConfig",
    "KimiVLConfig",
    "MoonViTConfig",
    "Step3VLConfig",
    "Step3TextConfig",
    "Step3VisionEncoderConfig",
    "Olmo3Config",
    "KimiLinearConfig",
    "KimiK25Config",
    "LagunaConfig",
    "Qwen3NextConfig",
    "Qwen3_5Config",
    "Qwen3_5MoeConfig",
    "InternS2PreviewConfig",
    "DotsVLMConfig",
    "DotsOCRConfig",
    "FalconH1Config",
    "GraniteMoeHybridConfig",
    "Lfm2Config",
    "Lfm2MoeConfig",
    "Lfm2VlConfig",
    "MiniCPMV4_6Config",
    "MiniCPMV4_6VisionConfig",
    "NemotronHConfig",
    "NemotronH_Nano_VL_V2_Config",
    "NemotronH_Nano_Omni_Reasoning_V3_Config",
    "JetNemotronConfig",
    "JetVLMConfig",
    "Step3p5Config",
    "Qwen3ASRConfig",
]
```
**EN:** This statement initializes __all__ in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 __all__。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

## Key Concepts / 关键概念
- **Structure / 结构**: The module is mostly composed of imports and lightweight wiring code. / 该模块主要由导入语句与轻量级衔接代码组成。

## Dependencies / 依赖关系
- **Local Modules / 本地模块**: `sglang.srt.configs.afmoe`, `sglang.srt.configs.bailing_hybrid`, `sglang.srt.configs.chatglm`, `sglang.srt.configs.dbrx`, `sglang.srt.configs.deepseekvl2`, `sglang.srt.configs.dots_ocr`, `sglang.srt.configs.dots_vlm`, `sglang.srt.configs.exaone`, `sglang.srt.configs.falcon_h1`, `sglang.srt.configs.granitemoehybrid`, `sglang.srt.configs.interns2preview`, `sglang.srt.configs.janus_pro`, `sglang.srt.configs.jet_nemotron`, `sglang.srt.configs.jet_vlm`, `sglang.srt.configs.kimi_k25`, `sglang.srt.configs.kimi_linear`, `sglang.srt.configs.kimi_vl`, `sglang.srt.configs.kimi_vl_moonvit`, `sglang.srt.configs.laguna`, `sglang.srt.configs.lfm2`, `sglang.srt.configs.lfm2_moe`, `sglang.srt.configs.lfm2_vl`, `sglang.srt.configs.longcat_flash`, `sglang.srt.configs.minicpmv4_6`, `sglang.srt.configs.nano_nemotron_vl`, `sglang.srt.configs.nemotron_h`, `sglang.srt.configs.olmo3`, `sglang.srt.configs.qwen3_5`, `sglang.srt.configs.qwen3_asr`, `sglang.srt.configs.qwen3_next`, `sglang.srt.configs.step3_vl`, `sglang.srt.configs.step3p5`

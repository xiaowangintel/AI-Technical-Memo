# config.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/dllm/config.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed LLM coordination part of the SRT runtime and implements logic centered on `config`. It exposes primary entry points such as `DllmConfig`. / 该模块属于 SRT 运行时的分布式 LLM 协调部分，主要实现围绕 `config` 的逻辑。 它对外提供的主要入口包括 `DllmConfig`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module imports, constants, and setup
```python
from typing import Any

from sglang.srt.configs.model_config import ModelConfig
from sglang.srt.server_args import ServerArgs


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 7-7: Class DllmConfig
```python
class DllmConfig:
```
**EN:** This range introduces `DllmConfig` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `DllmConfig`，并定义其后续方法依赖的结构或元数据。

### Lines 8-20: Method DllmConfig.__init__
```python
    def __init__(
        self,
        algorithm: str,
        algorithm_config: dict[str, Any],
        block_size: int,
        mask_id: int,
        max_running_requests: int,
    ):
        self.algorithm = algorithm
        self.algorithm_config = algorithm_config
        self.block_size = block_size
        self.mask_id = mask_id
        self.max_running_requests = max_running_requests
```
**EN:** This callable implements `DllmConfig.__init__`. It takes `algorithm`, `algorithm_config`, `block_size`, `mask_id` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `DllmConfig.__init__`。它接收 `algorithm`, `algorithm_config`, `block_size`, `mask_id`，主要用于初始化实例状态与默认值。

### Lines 22-75: Method DllmConfig.from_server_args
```python
    @staticmethod
    def from_server_args(
        server_args: ServerArgs,
    ):
        if server_args.dllm_algorithm is None:
            return None

        model_config = ModelConfig.from_server_args(
            server_args,
            model_path=server_args.model_path,
            model_revision=server_args.revision,
        )
        DLLM_PARAMS = {
            "LLaDA2MoeModelLM": {"block_size": 32, "mask_id": 156895},
            "SDARForCausalLM": {"block_size": 4, "mask_id": 151669},
            "SDARMoeForCausalLM": {"block_size": 4, "mask_id": 151669},
        }

        arch = model_config.hf_config.architectures[0]
        if arch in DLLM_PARAMS:
            params = DLLM_PARAMS[arch]
            block_size = params["block_size"]
            mask_id = params["mask_id"]
        else:
            raise RuntimeError(f"Unknown diffusion LLM: {arch}")

        max_running_requests = (
            1
            if server_args.max_running_requests is None
            else server_args.max_running_requests
        )

        algorithm_config = {}
        if server_args.dllm_algorithm_config is not None:
            try:
                import yaml
            except ImportError:
                raise ImportError(
                    "Please install PyYAML to use YAML config files. "
                    "`pip install pyyaml`"
                )
            with open(server_args.dllm_algorithm_config, "r") as f:
                algorithm_config = yaml.safe_load(f)

            # Parse common algorithm configurations
            block_size = algorithm_config.get("block_size", block_size)

        return DllmConfig(
            algorithm=server_args.dllm_algorithm,
            algorithm_config=algorithm_config,
            block_size=block_size,
            mask_id=mask_id,
            max_running_requests=max_running_requests,
        )
```
**EN:** This callable implements `DllmConfig.from_server_args`. It takes `server_args` and mainly constructs data from an external representation. In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `DllmConfig.from_server_args`。它接收 `server_args`，主要用于从外部表示构造数据。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

## Key Concepts / 关键概念
- `DllmConfig`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: `yaml`
- **Internal modules / 内部模块**: `sglang.srt.configs.model_config`, `sglang.srt.server_args`

# modelopt_quantize_and_export.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/usage/modelopt_quantize_and_export.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Example: ModelOpt Quantization and Export with SGLang This example demonstrates the streamlined workflow for quantizing a model with ModelOpt and automatically exporting it for deployment with SGLang. / 该文件的顶部说明概述了此示例的目标与使用场景。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Module overview
````python
#!/usr/bin/env python3
"""
Example: ModelOpt Quantization and Export with SGLang

This example demonstrates the streamlined workflow for quantizing a model with
ModelOpt and automatically exporting it for deployment with SGLang.
"""
````
**EN:** The opening docstring summarizes the goal of the file and provides high-level context for the code that follows.
**CN:** 开头的文档字符串概括了文件目标，并为后续代码提供高层背景。

### Lines 9-23: Import dependencies and runtime symbols
````python
import argparse
import os
from typing import Optional

import torch

import sglang as sgl
from sglang.srt.configs.device_config import DeviceConfig
from sglang.srt.configs.load_config import LoadConfig
from sglang.srt.configs.model_config import ModelConfig
from sglang.srt.distributed.parallel_state import (
    init_distributed_environment,
    initialize_model_parallel,
)
from sglang.srt.model_loader.loader import get_model_loader
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 26-53: Validate export
````python
def _validate_export(export_dir: str) -> bool:
    """Validate that an exported model directory contains the expected files."""
    import glob

    required_files = ["config.json", "tokenizer_config.json"]

    if not os.path.exists(export_dir):
        return False

    # Check required files
    for file in required_files:
        if not os.path.exists(os.path.join(export_dir, file)):
            return False

    # Check for model files using pattern matching to handle sharded models
    model_patterns = [
        "model*.safetensors",
        "pytorch_model*.bin",
    ]

    has_model_file = False
    for pattern in model_patterns:
        matching_files = glob.glob(os.path.join(export_dir, pattern))
        if matching_files:
            has_model_file = True
            break

    return has_model_file
````
**EN:** This function encapsulates the “Validate export” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Validate export”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 56-75: Get export info
````python
def _get_export_info(export_dir: str) -> Optional[dict]:
    """Get information about an exported model."""
    import json

    if not _validate_export(export_dir):
        return None

    try:
        config_path = os.path.join(export_dir, "config.json")
        with open(config_path, "r") as f:
            config = json.load(f)

        return {
            "model_type": config.get("model_type", "unknown"),
            "architectures": config.get("architectures", []),
            "quantization_config": config.get("quantization_config", {}),
            "export_dir": export_dir,
        }
    except Exception:
        return None
````
**EN:** This function encapsulates the “Get export info” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Get export info”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 78-172: Drive the engine runtime
````python
def quantize_and_export_model(
    model_path: str,
    export_dir: str,
    quantization_method: str = "modelopt_fp8",
    checkpoint_save_path: Optional[str] = None,
    device: str = "cuda",
) -> None:
    """
    Quantize a model with ModelOpt and export it for SGLang deployment.

    Args:
        model_path: Path to the original model
        export_dir: Directory to export the quantized model
        quantization_method: Quantization method ("modelopt_fp8" or "modelopt_fp4")
        checkpoint_save_path: Optional path to save ModelOpt checkpoint
        device: Device to use for quantization
    """
    print("🚀 Starting ModelOpt quantization and export workflow")
    print(f"📥 Input model: {model_path}")
    print(f"📤 Export directory: {export_dir}")
    print(f"⚙️  Quantization method: {quantization_method}")

    # Initialize minimal distributed environment for single GPU quantization
    if not torch.distributed.is_initialized():
        print("🔧 Initializing distributed environment...")
        # Set up environment variables for single-process distributed
        os.environ["RANK"] = "0"
        os.environ["WORLD_SIZE"] = "1"
        os.environ["MASTER_ADDR"] = "localhost"
        os.environ["MASTER_PORT"] = "12355"  # Use a different port than tests
        os.environ["LOCAL_RANK"] = "0"

        init_distributed_environment(
            world_size=1,
            rank=0,
            local_rank=0,
            backend="nccl" if device == "cuda" else "gloo",
        )
        initialize_model_parallel(
            tensor_model_parallel_size=1,
            pipeline_model_parallel_size=1,
        )

    # Configure model loading with ModelOpt quantization and export
    model_config = ModelConfig(
        model_path=model_path,
        quantization=quantization_method,  # Use unified quantization flag
        trust_remote_code=True,
    )

    load_config = LoadConfig(
        modelopt_checkpoint_save_path=checkpoint_save_path,
        modelopt_export_path=export_dir,
    )
    device_config = DeviceConfig(device=device)

    # Load and quantize the model (export happens automatically)
    print("🔄 Loading and quantizing model...")
    model_loader = get_model_loader(load_config, model_config)

    try:
        model_loader.load_model(
            model_config=model_config,
            device_config=device_config,
        )
        print("✅ Model quantized successfully!")

        # Validate the export
        if _validate_export(export_dir):
            print("✅ Export validation passed!")

            info = _get_export_info(export_dir)
            if info:
                print("📋 Model info:")
                print(f"   - Type: {info['model_type']}")
                print(f"   - Architecture: {info['architectures']}")
                print(f"   - Quantization: {info['quantization_config']}")
        else:
            print("❌ Export validation failed!")
            return

    except Exception as e:
        print(f"❌ Quantization failed: {e}")
        return

    print("\n🎉 Workflow completed successfully!")
    print(f"📁 Quantized model exported to: {export_dir}")
    print("\n🚀 To use the exported model:")
    print(
        f"   python -m sglang.launch_server --model-path {export_dir} --quantization modelopt"
    )
    print("\n   # Or in Python:")
    print("   import sglang as sgl")
    print(f"   llm = sgl.Engine(model_path='{export_dir}', quantization='modelopt')")
    print("   # Note: 'modelopt' auto-detects FP4/FP8 from model config")
````
**EN:** This function constructs or uses an SGLang engine instance and wraps a concrete runtime interaction around it.
**CN:** 该函数会构建或使用一个 SGLang 引擎实例，并围绕它封装具体的运行时交互。

### Lines 175-221: Drive the engine runtime
````python
def deploy_exported_model(
    export_dir: str,
    host: str = "127.0.0.1",
    port: int = 30000,
) -> None:
    """
    Deploy an exported ModelOpt quantized model with SGLang.

    Args:
        export_dir: Directory containing the exported model
        host: Host to bind the server to
        port: Port to bind the server to
    """
    print(f"🚀 Deploying exported model from: {export_dir}")

    # Validate export first
    if not _validate_export(export_dir):
        print("❌ Invalid export directory!")
        return

    try:
        # Launch SGLang engine with the exported model
        # Using generic "modelopt" for auto-detection of FP4/FP8
        llm = sgl.Engine(
            model_path=export_dir,
            quantization="modelopt",
            host=host,
            port=port,
        )

        print("✅ Model deployed successfully!")
        print(f"🌐 Server running at http://{host}:{port}")

        # Example inference
        prompts = ["Hello, how are you?", "What is the capital of France?"]
        sampling_params = {"temperature": 0.8, "top_p": 0.95, "max_new_tokens": 100}

        print("\n🧪 Running example inference...")
        outputs = llm.generate(prompts, sampling_params)

        for i, output in enumerate(outputs):
            print(f"Prompt {i+1}: {prompts[i]}")
            print(f"Output: {output['text']}")
            print()

    except Exception as e:
        print(f"❌ Deployment failed: {e}")
````
**EN:** This function constructs or uses an SGLang engine instance and wraps a concrete runtime interaction around it.
**CN:** 该函数会构建或使用一个 SGLang 引擎实例，并围绕它封装具体的运行时交互。

### Lines 224-299: Parse arguments and run the workflow
````python
def main():
    parser = argparse.ArgumentParser(
        description="ModelOpt Quantization and Export with SGLang",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  # Quantize and export a model (recommended workflow)
  python modelopt_quantize_and_export.py quantize \\
    --model-path TinyLlama/TinyLlama-1.1B-Chat-v1.0 \\
    --export-dir ./quantized_model \\
    --quantization-method modelopt_fp8

  # Deploy a pre-exported model
  python modelopt_quantize_and_export.py deploy \\
    --export-dir ./quantized_model
        """,
    )

    subparsers = parser.add_subparsers(dest="command", help="Available commands")

    # Quantize command
    quantize_parser = subparsers.add_parser(
        "quantize", help="Quantize and export a model"
    )
    quantize_parser.add_argument(
        "--model-path", required=True, help="Path to the model to quantize"
    )
    quantize_parser.add_argument(
        "--export-dir", required=True, help="Directory to export the quantized model"
    )
    quantize_parser.add_argument(
        "--quantization-method",
        choices=["modelopt_fp8", "modelopt_fp4"],
        default="modelopt_fp8",
        help="Quantization method to use",
    )
    quantize_parser.add_argument(
        "--checkpoint-save-path", help="Optional path to save ModelOpt checkpoint"
    )
    quantize_parser.add_argument(
        "--device", default="cuda", help="Device to use for quantization"
    )

    # TODO: Quantize-and-serve command removed due to compatibility issues
    # Use the separate quantize-then-deploy workflow instead

    # Deploy command
    deploy_parser = subparsers.add_parser("deploy", help="Deploy an exported model")
    deploy_parser.add_argument(
        "--export-dir", required=True, help="Directory containing the exported model"
    )
    deploy_parser.add_argument(
        "--host", default="127.0.0.1", help="Host to bind the server to"
    )
    deploy_parser.add_argument(
        "--port", type=int, default=30000, help="Port to bind the server to"
    )

    args = parser.parse_args()

    if args.command == "quantize":
        quantize_and_export_model(
            model_path=args.model_path,
            export_dir=args.export_dir,
            quantization_method=args.quantization_method,
            checkpoint_save_path=args.checkpoint_save_path,
            device=args.device,
        )
    elif args.command == "deploy":
        deploy_exported_model(
            export_dir=args.export_dir,
            host=args.host,
            port=args.port,
        )
    else:
        parser.print_help()
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 302-303: Program entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。
- **Structured generation / 结构化生成**: Schemas or constrained decoding keep outputs machine-readable. / 模式约束或受限解码让输出保持机器可读。
- **Distributed coordination / 分布式协同**: Multiple workers synchronize state, data, or updates. / 多个工作进程会同步状态、数据或更新。
- **Checkpoint management / 检查点管理**: Weights or runtime state are saved, loaded, or updated explicitly. / 权重或运行状态会被显式保存、加载或更新。
- **Quantization and export / 量化与导出**: The model is reduced and exported for efficient serving. / 模型会被压缩量化并导出，以便高效部署。

## Dependencies / 依赖关系
- **Standard library / 标准库**: argparse, glob, json, os, typing.Optional
- **Third-party / 第三方**: torch
- **Project-specific / 项目相关**: sglang, sglang.srt.configs.device_config.DeviceConfig, sglang.srt.configs.load_config.LoadConfig, sglang.srt.configs.model_config.ModelConfig, sglang.srt.distributed.parallel_state.init_distributed_environment, sglang.srt.distributed.parallel_state.initialize_model_parallel, sglang.srt.model_loader.loader.get_model_loader

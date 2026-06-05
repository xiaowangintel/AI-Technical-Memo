# test_maverick.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_maverick.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 1 test(s), 0 fixture(s), and 14 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 1 个测试、0 个 fixture，以及 14 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L13-L27)
```python
import json
import shutil
from pathlib import Path
from typing import Any

import pytest
import torch
from safetensors.torch import save_file
from transformers import AutoConfig, AutoProcessor, AutoTokenizer, GenerationConfig

from vllm import LLM, SamplingParams
from vllm.v1.executor.abstract import Executor
from vllm.v1.kv_cache_interface import ChunkedLocalAttentionSpec, FullAttentionSpec

from ....utils import multi_gpu_test
```
**EN:** Imports standard-library modules such as `argparse`, `json`, `pathlib.Path`, third-party packages like `pytest`, `safetensors.torch.save_file`, `torch`, project helpers such as `vllm.LLM`, `vllm.SamplingParams`, `vllm.v1.executor.abstract.Executor`.
**CN:** 导入标准库模块（如 `argparse`、`json`、`pathlib.Path`）、第三方包（如 `pytest`、`safetensors.torch.save_file`、`torch`）、项目内辅助模块（如 `vllm.LLM`、`vllm.SamplingParams`、`vllm.v1.executor.abstract.Executor`）。

### Module setup / 模块级配置: PROMPTS (L30-L35)
```python
PROMPTS: list[str] = [
    "Hello, my name is",
    "The president of the United States is",
    "The capital of France is",
    "The future of AI is",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `PROMPTS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `PROMPTS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: run_maverick_serving (L38-L70)
```python
def run_maverick_serving(model: str):
    """Test Llama-4-Maverick model with vLLM LLM class using CLI equivalent
    options with reduced layers.
    """

    try:
        sampling_params = SamplingParams(temperature=0.8, top_p=0.95)

        llm = LLM(
            model=model,
            max_model_len=2048,
            enforce_eager=True,
            tensor_parallel_size=8,
            enable_expert_parallel=True,
            trust_remote_code=True,
            gpu_memory_utilization=0.4,
            kv_cache_dtype="fp8",
        )
# ... 7 lines omitted for brevity ...
            generated_text = output.outputs[0].text
            print(f"Prompt:    {prompt!r}")
            print(f"Output:    {generated_text!r}")
            print("-" * 60)

    except Exception as e:
        print(f"Error initializing or running model: {e}")
        raise
```
**EN:** This helper encapsulates reusable logic in `run_maverick_serving`. Key inputs are `model`. It touches the core vLLM initialization or engine path directly.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_maverick_serving` 中。 关键输入包括 `model`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。

### Helper / 辅助函数: get_rope_layers_config (L73-L90)
```python
def get_rope_layers_config(model_path: str) -> list[int]:
    """
    Get the interleaved RoPE configuration from HuggingFace config

    Args:
        model_path: Path to the local directory containing the reduced
            Maverick model checkpoint

    Returns:
        List of 0 or 1 indicating whether each layer uses RoPE and local attn
        0 indicates that RoPE is not used while 1 indicates that RoPE is used.
    """
    config_path = Path(model_path) / "config.json"
    model_config = json.loads(config_path.read_text())
    text_config = model_config["text_config"]
    no_rope_layers = text_config["no_rope_layers"]
    print(f"Found no_rope_layers: {no_rope_layers}")
    return no_rope_layers
```
**EN:** This helper encapsulates reusable logic in `get_rope_layers_config`. Key inputs are `model_path`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_rope_layers_config` 中。 关键输入包括 `model_path`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: create_reduced_maverick_model (L93-L174)
```python
def create_reduced_maverick_model(
    original_model_name: str = "meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8",
    output_dir: str = "/tmp/reduced_maverick",
    text_layers: int = 4,
    num_experts: int = 4,
    vision_layers: int = 2,
    force_recreate: bool = False,
) -> str:
    """
    Create a reduced-layer version of the Maverick model.

    Args:
        original_model_name: Name of the original Maverick model
        output_dir: Directory to save the reduced model
        text_layers: Number of text transformer layers
        num_experts: Number of experts per layer
        vision_layers: Number of vision transformer layers
        force_recreate: Whether to recreate if output_dir already exists
# ... 56 lines omitted for brevity ...
        return str(output_path)

    except Exception as e:
        print(f"Error creating reduced model: {e}")
        # Clean up on failure
        if output_path.exists():
            shutil.rmtree(output_path)
        raise
```
**EN:** This helper encapsulates reusable logic in `create_reduced_maverick_model`. Key inputs are `original_model_name`, `output_dir`, `text_layers`, `num_experts`, `vision_layers`, `force_recreate`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `create_reduced_maverick_model` 中。 关键输入包括 `original_model_name`、`output_dir`、`text_layers`、`num_experts`、`vision_layers`、`force_recreate`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: create_reduced_config (L177-L218)
```python
def create_reduced_config(
    original_config: Any, text_layers: int, num_experts: int, vision_layers: int
) -> dict[str, Any]:
    """Create a reduced configuration based on the original."""

    # Convert config to dictionary
    config_dict = original_config.to_dict()

    # Reduce text layers
    if "text_config" in config_dict:
        original_text_layers = config_dict["text_config"]["num_hidden_layers"]
        config_dict["text_config"]["num_hidden_layers"] = text_layers
        original_layer_types = config_dict["text_config"]["layer_types"]
        config_dict["text_config"]["layer_types"] = original_layer_types[:text_layers]
        print(f"Reduced text layers from {original_text_layers} to {text_layers}")

        original_num_experts = config_dict["text_config"]["num_local_experts"]
        config_dict["text_config"]["num_local_experts"] = num_experts
# ... 16 lines omitted for brevity ...
        original_vision_layers = config_dict["vision_config"]["num_hidden_layers"]
        config_dict["vision_config"]["num_hidden_layers"] = vision_layers
        print(f"Reduced vision layers from {original_vision_layers} to {vision_layers}")

    # Update model name to indicate it's a reduced version
    config_dict["_name_or_path"] = f"reduced_maverick_{text_layers}t_{vision_layers}v"

    return config_dict
```
**EN:** This helper encapsulates reusable logic in `create_reduced_config`. Key inputs are `original_config`, `text_layers`, `num_experts`, `vision_layers`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `create_reduced_config` 中。 关键输入包括 `original_config`、`text_layers`、`num_experts`、`vision_layers`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: copy_tokenizer_files (L221-L231)
```python
def copy_tokenizer_files(original_model_name: str, output_path: Path) -> None:
    """Copy tokenizer files from the original model."""

    try:
        tokenizer = AutoTokenizer.from_pretrained(
            original_model_name, trust_remote_code=True
        )
        tokenizer.save_pretrained(output_path)
        print("Tokenizer files copied successfully")
    except Exception as e:
        print(f"Warning: Could not copy tokenizer files: {e}")
```
**EN:** This helper encapsulates reusable logic in `copy_tokenizer_files`. Key inputs are `original_model_name`, `output_path`.
**CN:** 这个辅助函数将可复用逻辑封装在 `copy_tokenizer_files` 中。 关键输入包括 `original_model_name`、`output_path`。

### Helper / 辅助函数: create_preprocessor_config (L234-L249)
```python
def create_preprocessor_config(original_config: Any, output_path: Path) -> None:
    """Create preprocessor_config.json for multimodal model."""

    # Try to load the original preprocessor config
    try:
        processor = AutoProcessor.from_pretrained(
            original_config._name_or_path
            or "meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8",
            trust_remote_code=True,
        )
        processor.save_pretrained(output_path)
        print("Copied original preprocessor config")
        return
    except Exception as e:
        print(f"Could not copy original preprocessor config: {e}")
        raise
```
**EN:** This helper encapsulates reusable logic in `create_preprocessor_config`. Key inputs are `original_config`, `output_path`.
**CN:** 这个辅助函数将可复用逻辑封装在 `create_preprocessor_config` 中。 关键输入包括 `original_config`、`output_path`。

### Helper / 辅助函数: create_reduced_safetensors (L252-L274)
```python
def create_reduced_safetensors(
    original_config: Any, reduced_config: dict[str, Any], output_path: Path
) -> None:
    """Create safetensors files with weights for the reduced model."""

    print("Generating synthetic weights for reduced model...")

    text_config = reduced_config["text_config"]
    vision_config = reduced_config["vision_config"]

    weights = {}

    print("Creating text model weights...")
    weights.update(create_text_model_weights(text_config))

    print("Creating vision model weights...")
    weights.update(create_vision_model_weights(vision_config))

    print("Creating shared model weights...")
    weights.update(create_shared_weights(text_config, vision_config))

    print("Saving weights to safetensors files...")
    save_weights_to_safetensors(weights, output_path)
```
**EN:** This helper encapsulates reusable logic in `create_reduced_safetensors`. Key inputs are `original_config`, `reduced_config`, `output_path`.
**CN:** 这个辅助函数将可复用逻辑封装在 `create_reduced_safetensors` 中。 关键输入包括 `original_config`、`reduced_config`、`output_path`。

### Helper / 辅助函数: create_text_model_weights (L277-L401)
```python
def create_text_model_weights(text_config: dict[str, Any]) -> dict[str, torch.Tensor]:
    """Create synthetic weights for the text model with MoE structure."""

    weights = {}

    vocab_size = text_config["vocab_size"]
    hidden_size = text_config["hidden_size"]
    intermediate_size = text_config["intermediate_size"]
    intermediate_size_mlp = text_config["intermediate_size_mlp"]
    num_layers = text_config["num_hidden_layers"]
    num_attention_heads = text_config["num_attention_heads"]
    num_key_value_heads = text_config.get("num_key_value_heads", num_attention_heads)

    # MoE specific parameters
    num_experts = text_config.get("num_local_experts")
    assert num_experts is not None, "num_local_experts must be specified for MoE"

    head_dim = hidden_size // num_attention_heads
# ... 99 lines omitted for brevity ...
    weights["language_model.model.norm.weight"] = torch.ones(
        hidden_size, dtype=torch.bfloat16
    )
    weights["language_model.lm_head.weight"] = torch.randn(
        vocab_size, hidden_size, dtype=torch.bfloat16
    )

    return weights
```
**EN:** This helper encapsulates reusable logic in `create_text_model_weights`. Key inputs are `text_config`. It returns computed state or helper objects back to the caller. The main assertion is `num_experts is not None`.
**CN:** 这个辅助函数将可复用逻辑封装在 `create_text_model_weights` 中。 关键输入包括 `text_config`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `num_experts is not None`。

### Helper / 辅助函数: create_vision_model_weights (L404-L470)
```python
def create_vision_model_weights(
    vision_config: dict[str, Any],
) -> dict[str, torch.Tensor]:
    """Create synthetic weights for the vision model."""

    weights = {}

    hidden_size = vision_config["hidden_size"]
    intermediate_size = vision_config["intermediate_size"]
    num_layers = vision_config["num_hidden_layers"]

    # Vision transformer layers
    for layer_idx in range(num_layers):
        layer_prefix = f"vision_model.model.layers.{layer_idx}"

        weights[f"{layer_prefix}.self_attn.q_proj.weight"] = torch.randn(
            hidden_size, hidden_size, dtype=torch.bfloat16
        )
# ... 41 lines omitted for brevity ...
        weights[f"{layer_prefix}.post_attention_layernorm.weight"] = torch.ones(
            hidden_size, dtype=torch.bfloat16
        )
        weights[f"{layer_prefix}.post_attention_layernorm.bias"] = torch.zeros(
            hidden_size, dtype=torch.bfloat16
        )

    return weights
```
**EN:** This helper encapsulates reusable logic in `create_vision_model_weights`. Key inputs are `vision_config`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `create_vision_model_weights` 中。 关键输入包括 `vision_config`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: create_shared_weights (L473-L488)
```python
def create_shared_weights(
    text_config: dict[str, Any], vision_config: dict[str, Any]
) -> dict[str, torch.Tensor]:
    """Create weights for shared components (vision-language connector)"""

    weights = {}

    text_hidden_size = text_config["hidden_size"]
    projector_input_dim = vision_config["projector_input_dim"]

    # Vision-language connector (projects vision features to text space)
    weights["multi_modal_projector.linear_1.weight"] = torch.randn(
        text_hidden_size, projector_input_dim, dtype=torch.bfloat16
    )

    return weights
```
**EN:** This helper encapsulates reusable logic in `create_shared_weights`. Key inputs are `text_config`, `vision_config`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `create_shared_weights` 中。 关键输入包括 `text_config`、`vision_config`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: save_weights_to_safetensors (L491-L553)
```python
def save_weights_to_safetensors(
    weights: dict[str, torch.Tensor], output_path: Path
) -> None:
    """Save weights to safetensors files and create index."""

    # Determine how to shard the weights
    max_shard_size = 5 * 1024 * 1024 * 1024  # 5GB per shard

    # Calculate sizes and create shards
    shards = []
    current_shard: dict[str, torch.Tensor] = {}
    current_size = 0

    for name, tensor in weights.items():
        tensor_size = tensor.numel() * tensor.element_size()

        if current_size + tensor_size > max_shard_size and current_shard:
            shards.append(current_shard)
# ... 37 lines omitted for brevity ...
    index_path = output_path / "model.safetensors.index.json"
    with open(index_path, "w") as f:
        json.dump(index_data, f, indent=2)

    print(f"Created index file: {index_path}")
    print(
        f"Total model size: {index_data['metadata']['total_size'] / (1024**3):.2f} GB"
    )
```
**EN:** This helper encapsulates reusable logic in `save_weights_to_safetensors`. Key inputs are `weights`, `output_path`.
**CN:** 这个辅助函数将可复用逻辑封装在 `save_weights_to_safetensors` 中。 关键输入包括 `weights`、`output_path`。

### Helper / 辅助函数: check_attention_spec_interleaved_rope (L556-L576)
```python
def check_attention_spec_interleaved_rope(
    llm: LLM,
    num_attention_layers: int,
    num_ranks: int,
    rope_layers: list[int],
):
    """Check that the attention spec is correct."""
    assert isinstance(llm.llm_engine.model_executor, Executor)
    kv_cache_specs_per_rank = llm.llm_engine.model_executor.get_kv_cache_specs()
    for rank in range(num_ranks):
        kv_cache_specs = kv_cache_specs_per_rank[rank]
        assert len(kv_cache_specs.keys()) == num_attention_layers
        for i in range(num_attention_layers):
            if rope_layers[i] == 0:
                expected_spec = FullAttentionSpec
            else:
                expected_spec = ChunkedLocalAttentionSpec
            assert isinstance(
                kv_cache_specs[f"language_model.model.layers.{i}.self_attn.attn"],
                expected_spec,
            )
```
**EN:** This helper encapsulates reusable logic in `check_attention_spec_interleaved_rope`. Key inputs are `llm`, `num_attention_layers`, `num_ranks`, `rope_layers`. The main assertion is `isinstance(llm.llm_engine.model_executor, Executor)` and `len(kv_cache_specs.keys()) == num_attention_layers`.
**CN:** 这个辅助函数将可复用逻辑封装在 `check_attention_spec_interleaved_rope` 中。 关键输入包括 `llm`、`num_attention_layers`、`num_ranks`、`rope_layers`。 核心断言是 `isinstance(llm.llm_engine.model_executor, Executor)` and `len(kv_cache_specs.keys()) == num_attention_layers`。

### Helper / 辅助函数: run_reduced_model (L579-L593)
```python
def run_reduced_model(llm: LLM, should_profile: bool = False) -> None:
    """Test the created reduced model with vLLM."""
    sampling_params = SamplingParams(temperature=0.8, top_p=0.95, max_tokens=50)

    if should_profile:
        llm.start_profile()
    outputs = llm.generate(PROMPTS, sampling_params)
    if should_profile:
        llm.stop_profile()

    print("Test generation successful!")
    for output in outputs:
        print(f"Prompt: {output.prompt}")
        print(f"Output: {output.outputs[0].text}")
        print("-" * 40)
```
**EN:** This helper encapsulates reusable logic in `run_reduced_model`. Key inputs are `llm`, `should_profile`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_reduced_model` 中。 关键输入包括 `llm`、`should_profile`。

### Test / 测试: test_dummy_maverick (L596-L651)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "original_model_name,text_layers,num_experts,vision_layers,",
    [("meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8", 4, 4, 2)],
)
@pytest.mark.parametrize("enforce_eager", [True, False])
@pytest.mark.parametrize("tp,ep", [(2, True)])
@pytest.mark.skipif(not torch.cuda.is_available(), reason="CUDA not available")
def test_dummy_maverick(
    monkeypatch,
    original_model_name: str,
    text_layers: int,
    num_experts: int,
    vision_layers: int,
    enforce_eager: bool,
    tp: int,
    ep: bool,
    output_dir: str = "/tmp/reduced_maverick",
# ... 30 lines omitted for brevity ...
        llm,
        text_layers,
        tp,
        rope_layers,
    )

    print(f"\nTesting reduced model at {model_path}...")
    run_reduced_model(llm=llm, should_profile=profile)
```
**EN:** This test validates `test_dummy_maverick`. It uses parameterization over `original_model_name`, `text_layers`, `num_experts`, `vision_layers`. Relevant pytest markers include `skipif`. Key inputs are `monkeypatch`, `original_model_name`, `text_layers`, `num_experts`, `vision_layers`, `enforce_eager`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It touches the core vLLM initialization or engine path directly.
**CN:** 这个测试验证 `test_dummy_maverick`。 它通过参数化组合 `original_model_name`、`text_layers`、`num_experts`、`vision_layers`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `monkeypatch`、`original_model_name`、`text_layers`、`num_experts`、`vision_layers`、`enforce_eager`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。

### Helper / 辅助函数: main (L654-L719)
```python
def main():
    """Main function to create and test the reduced model."""

    import argparse

    parser = argparse.ArgumentParser(
        description="Create a reduced-layer Maverick model"
    )
    parser.add_argument(
        "--output-dir",
        default="/tmp/reduced_maverick",
        help="Output directory for the reduced model",
    )
    parser.add_argument(
        "--text-layers",
        type=int,
        default=4,
        help="Number of text transformer layers",
# ... 40 lines omitted for brevity ...
            tp=2,
            ep=True,
            enforce_eager=True,
            profile=args.profile,
        )

    if args.test_original:
        run_maverick_serving(args.original_model)
```
**EN:** This helper encapsulates reusable logic in `main`.
**CN:** 这个辅助函数将可复用逻辑封装在 `main` 中。

### Module setup / 模块级配置: module state (L722-L723)
```python
if __name__ == "__main__":
    exit(main())
```
**EN:** This block performs module-level configuration such as conditional imports, environment checks, or shared setup logic.
**CN:** 该代码块执行模块级配置，例如条件导入、环境检查或共享初始化逻辑。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `argparse`, `json`, `pathlib.Path`, `shutil`, `typing.Any`
- **Third-party / 第三方**: `pytest`, `safetensors.torch.save_file`, `torch`, `transformers.AutoConfig`, `transformers.AutoProcessor`, `transformers.AutoTokenizer`, `transformers.GenerationConfig`
- **Project / 项目内**: `vllm.LLM`, `vllm.SamplingParams`, `vllm.v1.executor.abstract.Executor`, `vllm.v1.kv_cache_interface.ChunkedLocalAttentionSpec`, `vllm.v1.kv_cache_interface.FullAttentionSpec`
- **Local relative imports / 本地相对导入**: `....utils.multi_gpu_test`

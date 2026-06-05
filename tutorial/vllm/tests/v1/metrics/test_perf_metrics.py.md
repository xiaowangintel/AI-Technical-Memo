# test_perf_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/metrics/test_perf_metrics.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests for the analytic estimators in metrics/flops.py. / 该文件的文档字符串表明其用途：`tests for the analytic estimators in metrics/flops.py`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-5)
```python
"""
Tests for the analytic estimators in metrics/flops.py.
"""
```
**EN:** Module docstring that declares the scope of the file: Tests for the analytic estimators in metrics/flops.py.
**CN:** 模块文档字符串直接说明了文件范围：`tests for the analytic estimators in metrics/flops.py`。

### Imports and setup / 导入与设置 (lines 7-34)
```python
import types
from types import SimpleNamespace

import pytest
from transformers.models.deepseek_v3.configuration_deepseek_v3 import DeepseekV3Config
from transformers.models.llama4.configuration_llama4 import (
    Llama4Config,
    Llama4TextConfig,
)
from transformers.models.qwen3.configuration_qwen3 import Qwen3Config
from transformers.models.qwen3_moe.configuration_qwen3_moe import Qwen3MoeConfig

from vllm.config.model import ModelConfig, get_hf_text_config
from vllm.transformers_utils.model_arch_config_convertor import (
    MODEL_ARCH_CONFIG_CONVERTORS,
    ModelArchConfigConvertorBase,
)
from vllm.v1.metrics.perf import (
    _QUANT_WEIGHT_BYTE_SIZE,
    AttentionMetrics,
    BaseConfigParser,
    ExecutionContext,
    FfnMetrics,
    InvalidComponent,
    ModelMetrics,
    ParsedArgs,
    UnembedMetrics,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, transformers.models.deepseek_v3.configuration_deepseek_v3, transformers.models.llama4.configuration_llama4, transformers.models.qwen3.configuration_qwen3, transformers.models.qwen3_moe.configuration_qwen3_moe`. vLLM modules under test include `vllm.config.model, vllm.transformers_utils.model_arch_config_convertor, vllm.v1.metrics.perf`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, transformers.models.deepseek_v3.configuration_deepseek_v3, transformers.models.llama4.configuration_llama4, transformers.models.qwen3.configuration_qwen3, transformers.models.qwen3_moe.configuration_qwen3_moe`。 被测试的 vLLM 模块包括 `vllm.config.model, vllm.transformers_utils.model_arch_config_convertor, vllm.v1.metrics.perf`。

### MockModelConfig (lines 37-76)
```python
class MockModelConfig:
    """Mock ModelConfig that implements the getter methods used by parsers."""

    def __init__(self, hf_config, dtype):
        self.hf_config = hf_config
        self.hf_text_config = get_hf_text_config(hf_config)
        convertor_cls = MODEL_ARCH_CONFIG_CONVERTORS.get(
            self.hf_config.model_type, ModelArchConfigConvertorBase
        )
        self.model_arch_config = convertor_cls(
            self.hf_config, self.hf_text_config
        ).convert()
        self.dtype = dtype
        self.is_attention_free = False
    def __getattr__(self, name):
        # 1. Check if ModelConfig actually has this attribute
        if not hasattr(ModelConfig, name):
    # ... excerpt omitted for brevity ...
            return attr.__get__(self, self.__class__)
            # this instance. This creates a bound method where 'self' is
            # automatically passed as the first arg.
            return types.MethodType(attr, self)
        # 5. Case C: It is a class attribute / static variable
        return attr
```
**EN:** Class `MockModelConfig` groups 0 test method(s) and 2 helper/fixture method(s).
**CN:** 类 `MockModelConfig` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。

### create_mock_vllm_config (lines 79-103)
```python
def create_mock_vllm_config(
    hf_config,
    model_dtype="bfloat16",
    cache_dtype="auto",
    quant_config=None,
    data_parallel_size=1,
    tensor_parallel_size=1,
    pipeline_parallel_size=1,
    enable_expert_parallel=False,
) -> SimpleNamespace:
    vllm_config = SimpleNamespace()
    vllm_config.model_config = MockModelConfig(hf_config, model_dtype)

    vllm_config.cache_config = SimpleNamespace()
    vllm_config.cache_config.cache_dtype = cache_dtype

    vllm_config.quant_config = quant_config

    vllm_config.parallel_config = SimpleNamespace()
    vllm_config.parallel_config.data_parallel_size = data_parallel_size
    vllm_config.parallel_config.tensor_parallel_size = tensor_parallel_size
    vllm_config.parallel_config.pipeline_parallel_size = pipeline_parallel_size
    vllm_config.parallel_config.enable_expert_parallel = enable_expert_parallel

    return vllm_config
```
**EN:** Helper function `create_mock_vllm_config` encapsulates reusable logic for `mock vllm config`. Inputs: `hf_config, model_dtype, cache_dtype, quant_config, data_parallel_size, tensor_parallel_size, pipeline_parallel_size, enable_expert_parallel`. Key calls include `SimpleNamespace, MockModelConfig`.
**CN:** 辅助函数 `create_mock_vllm_config` 封装了与 `mock vllm config` 相关的可复用逻辑。 输入参数：`hf_config, model_dtype, cache_dtype, quant_config, data_parallel_size, tensor_parallel_size, pipeline_parallel_size, enable_expert_parallel`。 关键调用包括 `SimpleNamespace, MockModelConfig`。

### test_base_config_parser (lines 109-128)
```python
def test_base_config_parser():
    """Test BaseConfigParser extracts base model attributes correctly."""
    hf_config = Qwen3Config(
        vocab_size=50000,
        hidden_size=2048,
        num_attention_heads=16,
        num_hidden_layers=24,
    )
    vllm_config = create_mock_vllm_config(hf_config, model_dtype="float16")

    parser = BaseConfigParser()
    args = ParsedArgs()
    result = parser.parse(args, vllm_config)

    assert result.vocab_size == 50000
    assert result.hidden_size == 2048
    assert result.num_attention_heads == 16
    assert result.num_hidden_layers == 24
    assert result.weight_byte_size == 2  # float16 is 2 bytes
    assert result.activation_byte_size == 2  # default activation size
```
**EN:** Test case covering `base config parser`. It exercises `Qwen3Config, create_mock_vllm_config, BaseConfigParser, ParsedArgs, parser.parse`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `base config parser` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, BaseConfigParser, ParsedArgs, parser.parse`。 代码主体包含 6 个显式断言。

### test_base_attention_config_parser_with_gqa (lines 131-145)
```python
def test_base_attention_config_parser_with_gqa():
    """Test BaseAttentionConfigParser with grouped query attention."""
    hf_config = Qwen3Config(
        hidden_size=4096,
        num_attention_heads=32,
        num_key_value_heads=8,  # GQA with 4:1 ratio
        head_dim=128,
    )
    vllm_config = create_mock_vllm_config(hf_config)

    parser_chain = AttentionMetrics.get_parser()
    result = parser_chain.parse(vllm_config)

    assert result.num_key_value_heads == 8
    assert result.head_dim == 128
```
**EN:** Test case covering `base attention config parser with gqa`. It exercises `Qwen3Config, create_mock_vllm_config, AttentionMetrics.get_parser, parser_chain.parse`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `base 注意力 config parser with gqa` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, AttentionMetrics.get_parser, parser_chain.parse`。 代码主体包含 2 个显式断言。

### test_base_attention_config_parser_without_gqa (lines 148-164)
```python
def test_base_attention_config_parser_without_gqa():
    """
    Test BaseAttentionConfigParser defaults to MHA when num_key_value_heads not
    specified.
    """
    hf_config = Qwen3Config(
        hidden_size=4096,
        num_attention_heads=32,
        # No num_key_value_heads specified
    )
    vllm_config = create_mock_vllm_config(hf_config)

    parser_chain = AttentionMetrics.get_parser()
    result = parser_chain.parse(vllm_config)

    # Should default to MHA (num_key_value_heads = num_attention_heads)
    assert result.num_key_value_heads == 32
```
**EN:** Test case covering `base attention config parser without gqa`. It exercises `Qwen3Config, create_mock_vllm_config, AttentionMetrics.get_parser, parser_chain.parse`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `base 注意力 config parser without gqa` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, AttentionMetrics.get_parser, parser_chain.parse`。 代码主体包含 1 个显式断言。

### test_base_ffn_config_parser_dense (lines 167-182)
```python
def test_base_ffn_config_parser_dense():
    """Test BaseFfnConfigParser for dense FFN."""
    hf_config = Qwen3Config(
        hidden_size=4096,
        intermediate_size=11008,
        num_hidden_layers=32,
    )
    vllm_config = create_mock_vllm_config(hf_config)

    parser_chain = FfnMetrics.get_parser()
    result = parser_chain.parse(vllm_config)

    assert result.intermediate_size == 11008
    assert result.num_experts == 0
    assert result.num_experts_per_tok == 0
    assert result.num_moe_layers == 0  # No MoE
```
**EN:** Test case covering `base ffn config parser dense`. It exercises `Qwen3Config, create_mock_vllm_config, FfnMetrics.get_parser, parser_chain.parse`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `base ffn config parser dense` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, FfnMetrics.get_parser, parser_chain.parse`。 代码主体包含 4 个显式断言。

### test_base_ffn_config_parser_moe (lines 185-205)
```python
def test_base_ffn_config_parser_moe():
    """Test BaseFfnConfigParser for MoE FFN."""
    hf_config = Qwen3MoeConfig(
        hidden_size=4096,
        intermediate_size=11008,
        num_hidden_layers=32,
        num_experts=64,
        num_experts_per_tok=8,
        moe_intermediate_size=14336,
        n_shared_experts=2,
    )
    vllm_config = create_mock_vllm_config(hf_config)

    parser_chain = FfnMetrics.get_parser()
    result = parser_chain.parse(vllm_config)

    assert result.num_experts == 64
    assert result.num_experts_per_tok == 8
    assert result.moe_intermediate_size == 14336
    assert result.num_shared_experts == 2
    assert result.num_moe_layers == 32  # All layers are MoE by default
```
**EN:** Test case covering `base ffn config parser moe`. It exercises `Qwen3MoeConfig, create_mock_vllm_config, FfnMetrics.get_parser, parser_chain.parse`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `base ffn config parser moe` 的测试用例。 该测试会调用 `Qwen3MoeConfig, create_mock_vllm_config, FfnMetrics.get_parser, parser_chain.parse`。 代码主体包含 5 个显式断言。

### test_interleave_moe_layer_step_parser (lines 208-223)
```python
def test_interleave_moe_layer_step_parser():
    """Test InterleaveMoeLayerStepParser correctly computes MoE layer count."""
    hf_config = Llama4Config(
        text_config=Llama4TextConfig(
            num_hidden_layers=32,
            num_local_experts=64,
            interleave_moe_layer_step=4,  # Every 4th layer is MoE
        ),
    )

    vllm_config = create_mock_vllm_config(hf_config)

    parser_chain = FfnMetrics.get_parser()
    result = parser_chain.parse(vllm_config)

    assert result.num_moe_layers == 8
```
**EN:** Test case covering `interleave moe layer step parser`. It exercises `Llama4Config, create_mock_vllm_config, FfnMetrics.get_parser, parser_chain.parse, Llama4TextConfig`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `interleave moe layer step parser` 的测试用例。 该测试会调用 `Llama4Config, create_mock_vllm_config, FfnMetrics.get_parser, parser_chain.parse, Llama4TextConfig`。 代码主体包含 1 个显式断言。

### test_moe_layer_freq_parser (lines 226-244)
```python
def test_moe_layer_freq_parser():
    """Test MoeLayerFreqParser correctly computes MoE layer count."""
    hf_config = DeepseekV3Config(
        num_hidden_layers=30,
        n_routed_experts=64,
        moe_layer_freq=3,  # Every 3rd layer after first_k_dense_replace
        first_k_dense_replace=6,  # First 6 layers are dense
    )
    vllm_config = create_mock_vllm_config(hf_config)

    parser_chain = FfnMetrics.get_parser()
    result = parser_chain.parse(vllm_config)

    # Layers >= 6 and divisible by 3: 6, 9, 12, 15, 18, 21, 24, 27
    expected_moe_layers = len(
        [layer for layer in range(30) if layer >= 6 and layer % 3 == 0]
    )
    assert expected_moe_layers == 8
    assert result.num_moe_layers == expected_moe_layers
```
**EN:** Test case covering `moe layer freq parser`. It exercises `DeepseekV3Config, create_mock_vllm_config, FfnMetrics.get_parser, parser_chain.parse, len, range`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `moe layer freq parser` 的测试用例。 该测试会调用 `DeepseekV3Config, create_mock_vllm_config, FfnMetrics.get_parser, parser_chain.parse, len, range`。 代码主体包含 2 个显式断言。

### test_attention_metrics_scaling (lines 250-290)
```python
def test_attention_metrics_scaling():
    """Test that attention metrics scale proportionally with model dimensions."""
    base_hf_config = Qwen3Config(
        hidden_size=2048,
        num_attention_heads=16,
        num_key_value_heads=16,
        num_hidden_layers=12,
        head_dim=128,
    )

    base_vllm_config = create_mock_vllm_config(base_hf_config)
    base_metrics = AttentionMetrics.from_vllm_config(base_vllm_config)
    # Test scaling with number of layers
    double_layers_hf_config = Qwen3Config(
    # ... excerpt omitted for brevity ...
    assert double_flops == 2 * base_flops
    double_read = double_layers_metrics.get_read_bytes(ctx)
    assert double_read == 2 * base_read
    base_write = base_metrics.get_write_bytes(ctx)
    double_write = double_layers_metrics.get_write_bytes(ctx)
    assert double_write == 2 * base_write
```
**EN:** Test case covering `attention metrics scaling`. It exercises `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, base_metrics.get_num_flops, double_layers_metrics.get_num_flops`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `注意力 metrics scaling` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, base_metrics.get_num_flops, double_layers_metrics.get_num_flops`。 代码主体包含 3 个显式断言。

### test_attention_metrics_grouped_query (lines 293-321)
```python
def test_attention_metrics_grouped_query():
    """Test attention metrics handle grouped query attention correctly."""
    mha_hf_config = Qwen3Config(
        hidden_size=4096,
        num_attention_heads=32,
        num_key_value_heads=32,  # MHA
        num_hidden_layers=1,
    )
    mha_config = create_mock_vllm_config(mha_hf_config)

    gqa_hf_config = Qwen3Config(
        hidden_size=4096,
        num_attention_heads=32,
        num_key_value_heads=8,  # GQA with 4:1 ratio
        num_hidden_layers=1,
    )
    gqa_config = create_mock_vllm_config(gqa_hf_config)

    mha_metrics = AttentionMetrics.from_vllm_config(mha_config)
    gqa_metrics = AttentionMetrics.from_vllm_config(gqa_config)

    ctx = ExecutionContext.from_single_request(
        num_tokens=1, context_len=1024, is_prefill=False
    )

    # GQA should have less KV cache reads since fewer KV heads
    mha_read = mha_metrics.get_read_bytes(ctx)
    gqa_read = gqa_metrics.get_read_bytes(ctx)
    assert gqa_read < mha_read
```
**EN:** Test case covering `attention metrics grouped query`. It exercises `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, mha_metrics.get_read_bytes, gqa_metrics.get_read_bytes`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `注意力 metrics grouped query` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, mha_metrics.get_read_bytes, gqa_metrics.get_read_bytes`。 代码主体包含 1 个显式断言。

### test_ffn_metrics_scaling (lines 324-350)
```python
def test_ffn_metrics_scaling():
    """Test FFN metrics scale proportionally with model dimensions."""
    base_hf_config = Qwen3Config(
        hidden_size=2048,
        intermediate_size=8192,
        num_hidden_layers=12,
    )
    base_vllm_config = create_mock_vllm_config(base_hf_config)
    base_metrics = FfnMetrics.from_vllm_config(base_vllm_config)

    # Test scaling with intermediate size
    larger_ffn_hf_config = Qwen3Config(
        hidden_size=2048,
        intermediate_size=16384,  # Double intermediate size
        num_hidden_layers=12,
    )
    larger_ffn_vllm_config = create_mock_vllm_config(larger_ffn_hf_config)
    larger_ffn_metrics = FfnMetrics.from_vllm_config(larger_ffn_vllm_config)

    ctx = ExecutionContext.from_single_request(
        num_tokens=100, context_len=512, is_prefill=True
    )

    # FLOPS should double when intermediate size doubles
    base_flops = base_metrics.get_num_flops(ctx)
    larger_flops = larger_ffn_metrics.get_num_flops(ctx)
    assert larger_flops == base_flops * 2
```
**EN:** Test case covering `ffn metrics scaling`. It exercises `Qwen3Config, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, base_metrics.get_num_flops, larger_ffn_metrics.get_num_flops`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `ffn metrics scaling` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, base_metrics.get_num_flops, larger_ffn_metrics.get_num_flops`。 代码主体包含 1 个显式断言。

### test_moe_metrics_vs_dense (lines 353-385)
```python
def test_moe_metrics_vs_dense():
    """Test MoE metrics versus dense metrics."""
    dense_hf_config = Qwen3Config(
        hidden_size=2048,
        intermediate_size=8192,
        num_hidden_layers=12,
    )
    dense_config = create_mock_vllm_config(dense_hf_config)

    moe_hf_config = Qwen3MoeConfig(
        num_experts=64,
        num_experts_per_tok=2,  # 2 routed expert
        moe_intermediate_size=8192,
        n_shared_experts=0,
    # ... excerpt omitted for brevity ...
    # MoE should have different compute/memory characteristics
    dense_flops = dense_metrics.get_num_flops(ctx)
    moe_flops = moe_metrics.get_num_flops(ctx)
    # 2 routed experts vs 1 dense.
    assert moe_flops == dense_flops * 2
```
**EN:** Test case covering `moe metrics vs dense`. It exercises `Qwen3Config, create_mock_vllm_config, Qwen3MoeConfig, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, dense_metrics.get_num_flops`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `moe metrics vs dense` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, Qwen3MoeConfig, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, dense_metrics.get_num_flops`。 代码主体包含 1 个显式断言。

### test_unembed_metrics_scaling (lines 388-412)
```python
def test_unembed_metrics_scaling():
    """Test unembedding metrics scale with vocab size."""
    small_vocab_hf_config = Qwen3Config(
        hidden_size=2048,
        vocab_size=32000,
    )
    small_vocab_config = create_mock_vllm_config(small_vocab_hf_config)

    large_vocab_hf_config = Qwen3Config(
        hidden_size=2048,
        vocab_size=64000,  # Double vocab size
    )
    large_vocab_config = create_mock_vllm_config(large_vocab_hf_config)

    small_vocab_metrics = UnembedMetrics.from_vllm_config(small_vocab_config)
    large_vocab_metrics = UnembedMetrics.from_vllm_config(large_vocab_config)

    ctx = ExecutionContext.from_single_request(
        num_tokens=100, context_len=512, is_prefill=True
    )

    # FLOPS should double when vocab size doubles
    small_flops = small_vocab_metrics.get_num_flops(ctx)
    large_flops = large_vocab_metrics.get_num_flops(ctx)
    assert large_flops == 2 * small_flops
```
**EN:** Test case covering `unembed metrics scaling`. It exercises `Qwen3Config, create_mock_vllm_config, UnembedMetrics.from_vllm_config, ExecutionContext.from_single_request, small_vocab_metrics.get_num_flops, large_vocab_metrics.get_num_flops`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `unembed metrics scaling` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, UnembedMetrics.from_vllm_config, ExecutionContext.from_single_request, small_vocab_metrics.get_num_flops, large_vocab_metrics.get_num_flops`。 代码主体包含 1 个显式断言。

### test_prefill_vs_decode_differences (lines 415-437)
```python
def test_prefill_vs_decode_differences():
    """Test that prefill and decode have different memory access patterns."""
    hf_config = Qwen3Config(
        hidden_size=2048,
        num_attention_heads=16,
        num_key_value_heads=16,
        num_hidden_layers=1,
    )
    config = create_mock_vllm_config(hf_config)

    metrics = AttentionMetrics.from_vllm_config(config)

    prefill_ctx = ExecutionContext.from_single_request(
        num_tokens=512, context_len=512, is_prefill=True
    )
    decode_ctx = ExecutionContext.from_single_request(
        num_tokens=1, context_len=512, is_prefill=False
    )

    prefill_read = metrics.get_read_bytes(prefill_ctx)
    decode_read = metrics.get_read_bytes(decode_ctx)

    assert prefill_read != decode_read
```
**EN:** Test case covering `prefill vs decode differences`. It exercises `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_read_bytes`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill vs decode differences` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_read_bytes`。 代码主体包含 1 个显式断言。

### test_model_metrics_aggregation (lines 440-461)
```python
def test_model_metrics_aggregation():
    """Test ModelMetrics correctly aggregates across components."""
    hf_config = Qwen3Config(
        hidden_size=2048,
        num_attention_heads=16,
        num_hidden_layers=12,
        vocab_size=32000,
        intermediate_size=8192,
    )
    config = create_mock_vllm_config(hf_config)

    model_metrics = ModelMetrics(config)
    ctx = ExecutionContext.from_single_request(
        num_tokens=100, context_len=512, is_prefill=True
    )

    # Should have metrics for attention, ffn, and unembed
    total_flops = model_metrics.get_num_flops(ctx)
    breakdown = model_metrics.get_num_flops_breakdown(ctx)

    # Breakdown should sum to total
    assert total_flops == sum(breakdown.values())
```
**EN:** Test case covering `model metrics aggregation`. It exercises `Qwen3Config, create_mock_vllm_config, ModelMetrics, ExecutionContext.from_single_request, model_metrics.get_num_flops, model_metrics.get_num_flops_breakdown`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `model metrics aggregation` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, ModelMetrics, ExecutionContext.from_single_request, model_metrics.get_num_flops, model_metrics.get_num_flops_breakdown`。 代码主体包含 1 个显式断言。

### test_moe_expert_activation_proportional_scaling (lines 464-540)
```python
def test_moe_expert_activation_proportional_scaling():
    """Test that routed expert metrics scale proportionally with num_experts_per_tok."""
    base_moe_config = Qwen3MoeConfig(
        hidden_size=2048,
        intermediate_size=8192,
        num_hidden_layers=12,
        num_experts=64,
        num_experts_per_tok=1,  # 1 expert per token
        moe_intermediate_size=8192,
        n_shared_experts=2,
    )

    double_experts_config = Qwen3MoeConfig(
        num_experts_per_tok=2,  # 2 experts per token (double)
    # ... excerpt omitted for brevity ...
    assert two_expert_diff == 2 * one_expert_diff
    assert two_expert_read_diff == 2 * one_expert_read_diff
    triple_write = triple_metrics.get_write_bytes(ctx)
    one_expert_write_diff = double_write - base_write
    two_expert_write_diff = triple_write - base_write
    assert two_expert_write_diff == 2 * one_expert_write_diff
```
**EN:** Test case covering `moe expert activation proportional scaling`. It exercises `Qwen3MoeConfig, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, base_metrics.get_num_flops, double_metrics.get_num_flops`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `moe expert activation proportional scaling` 的测试用例。 该测试会调用 `Qwen3MoeConfig, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, base_metrics.get_num_flops, double_metrics.get_num_flops`。 代码主体包含 3 个显式断言。

### test_quantization_config_parser_fp8 (lines 543-559)
```python
def test_quantization_config_parser_fp8():
    """Test quantization parsers with fp8."""

    class MockQuantConfig:
        def get_name(self):
            return "fp8"

    hf_config = Qwen3Config(
        hidden_size=2048, num_attention_heads=16, num_hidden_layers=1
    )
    vllm_config = create_mock_vllm_config(hf_config, quant_config=MockQuantConfig())

    attn_result = AttentionMetrics.get_parser().parse(vllm_config)
    assert attn_result.weight_byte_size == 1  # fp8

    ffn_result = FfnMetrics.get_parser().parse(vllm_config)
    assert ffn_result.weight_byte_size == 1  # fp8
```
**EN:** Test case covering `quantization config parser fp8`. It exercises `Qwen3Config, create_mock_vllm_config, get_parser.parse, MockQuantConfig, AttentionMetrics.get_parser, FfnMetrics.get_parser`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `quantization config parser fp8` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, get_parser.parse, MockQuantConfig, AttentionMetrics.get_parser, FfnMetrics.get_parser`。 代码主体包含 2 个显式断言。

### test_quantization_config_parser_mxfp4 (lines 562-575)
```python
def test_quantization_config_parser_mxfp4():
    """Test quantization parsers with mxfp4."""

    class MockQuantConfig:
        def get_name(self):
            return "mxfp4"

    hf_config = Qwen3Config(
        hidden_size=2048, intermediate_size=8192, num_hidden_layers=1
    )
    vllm_config = create_mock_vllm_config(hf_config, quant_config=MockQuantConfig())

    ffn_result = FfnMetrics.get_parser().parse(vllm_config)
    assert ffn_result.weight_byte_size == 0.5  # mxfp4
```
**EN:** Test case covering `quantization config parser mxfp4`. It exercises `Qwen3Config, create_mock_vllm_config, get_parser.parse, MockQuantConfig, FfnMetrics.get_parser`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `quantization config parser mxfp4` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, get_parser.parse, MockQuantConfig, FfnMetrics.get_parser`。 代码主体包含 1 个显式断言。

### test_attention_per_gpu_with_tensor_parallelism (lines 581-613)
```python
def test_attention_per_gpu_with_tensor_parallelism():
    """Test attention metrics with tensor parallelism - per_gpu vs global."""
    hf_config = Qwen3Config(
        hidden_size=4096,
        num_attention_heads=32,
        num_key_value_heads=8,
        num_hidden_layers=24,
    )

    # Test with TP=4
    vllm_config = create_mock_vllm_config(hf_config, tensor_parallel_size=4)
    metrics = AttentionMetrics.from_vllm_config(vllm_config)
    ctx = ExecutionContext.from_single_request(
        num_tokens=128, context_len=1024, is_prefill=True
    # Get global and per-gpu metrics
    # ... excerpt omitted for brevity ...
    assert global_flops == 4 * per_gpu_flops
    # Reads should scale similarly (weight reads are divided by TP)
    assert global_read > per_gpu_read
    global_write = metrics.get_write_bytes(ctx, per_gpu=False)
    per_gpu_write = metrics.get_write_bytes(ctx, per_gpu=True)
    assert global_write > per_gpu_write
```
**EN:** Test case covering `attention per GPU with tensor parallelism`. It exercises `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops, metrics.get_read_bytes`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `注意力 per gpu with tensor parallelism` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops, metrics.get_read_bytes`。 代码主体包含 3 个显式断言。

### test_attention_per_gpu_with_pipeline_parallelism (lines 616-641)
```python
def test_attention_per_gpu_with_pipeline_parallelism():
    """Test attention metrics with pipeline parallelism - per_gpu vs global."""
    hf_config = Qwen3Config(
        hidden_size=2048,
        num_attention_heads=16,
        num_hidden_layers=32,
    )

    # Test with PP=4
    vllm_config = create_mock_vllm_config(hf_config, pipeline_parallel_size=4)
    metrics = AttentionMetrics.from_vllm_config(vllm_config)

    ctx = ExecutionContext.from_single_request(
        num_tokens=100, context_len=512, is_prefill=False
    )

    # Get global and per-gpu metrics
    global_flops = metrics.get_num_flops(ctx, per_gpu=False)
    per_gpu_flops = metrics.get_num_flops(ctx, per_gpu=True)

    # With PP=4, global flops should be 4x per-gpu flops (layers divided by 4)
    assert global_flops == 4 * per_gpu_flops

    global_read = metrics.get_read_bytes(ctx, per_gpu=False)
    per_gpu_read = metrics.get_read_bytes(ctx, per_gpu=True)
    assert global_read == 4 * per_gpu_read
```
**EN:** Test case covering `attention per GPU with pipeline parallelism`. It exercises `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops, metrics.get_read_bytes`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `注意力 per gpu with pipeline parallelism` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops, metrics.get_read_bytes`。 代码主体包含 2 个显式断言。

### test_ffn_per_gpu_with_tensor_parallelism (lines 644-672)
```python
def test_ffn_per_gpu_with_tensor_parallelism():
    """Test FFN metrics with tensor parallelism - per_gpu vs global."""
    hf_config = Qwen3Config(
        hidden_size=4096,
        intermediate_size=14336,
        num_hidden_layers=32,
    )

    # Test with DP=2, TP=4 (ffn_tp_size will be 8)
    vllm_config = create_mock_vllm_config(
        hf_config,
        data_parallel_size=2,
        tensor_parallel_size=4,
    )
    metrics = FfnMetrics.from_vllm_config(vllm_config)

    # ffn_tp_size should be dp_size * tp_size = 8 (when EP not enabled)
    assert metrics.ffn_tp_size == 8

    ctx = ExecutionContext.from_single_request(
        num_tokens=128, context_len=2048, is_prefill=True
    )

    # Get global and per-gpu metrics
    global_flops = metrics.get_num_flops(ctx, per_gpu=False)
    per_gpu_flops = metrics.get_num_flops(ctx, per_gpu=True)

    # With ffn_tp_size=8, global should be 8x per-gpu
    assert global_flops == 8 * per_gpu_flops
```
**EN:** Test case covering `ffn per GPU with tensor parallelism`. It exercises `Qwen3Config, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `ffn per gpu with tensor parallelism` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops`。 代码主体包含 2 个显式断言。

### test_ffn_per_gpu_with_pipeline_parallelism (lines 675-696)
```python
def test_ffn_per_gpu_with_pipeline_parallelism():
    """Test FFN metrics with pipeline parallelism - per_gpu vs global."""
    hf_config = Qwen3Config(
        hidden_size=2048,
        intermediate_size=8192,
        num_hidden_layers=24,
    )

    # Test with PP=6
    vllm_config = create_mock_vllm_config(hf_config, pipeline_parallel_size=6)
    metrics = FfnMetrics.from_vllm_config(vllm_config)

    ctx = ExecutionContext.from_single_request(
        num_tokens=100, context_len=512, is_prefill=True
    )

    # Get global and per-gpu metrics
    global_flops = metrics.get_num_flops(ctx, per_gpu=False)
    per_gpu_flops = metrics.get_num_flops(ctx, per_gpu=True)

    # With PP=6, global should be 6x per-gpu (layers divided by 6)
    assert global_flops == 6 * per_gpu_flops
```
**EN:** Test case covering `ffn per GPU with pipeline parallelism`. It exercises `Qwen3Config, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `ffn per gpu with pipeline parallelism` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops`。 代码主体包含 1 个显式断言。

### test_moe_per_gpu_with_expert_parallelism (lines 699-754)
```python
def test_moe_per_gpu_with_expert_parallelism():
    """
    Test MoE metrics with expert parallelism - verifies num_activated_experts bug fix.
    hf_config = Qwen3MoeConfig(
        hidden_size=2048,
        intermediate_size=8192,
        num_hidden_layers=24,
        num_experts=64,
        num_experts_per_tok=8,
        moe_intermediate_size=14336,
        n_shared_experts=2,
    )

    # Test with DP=2, TP=4, EP enabled (ffn_ep_size will be 8)
    vllm_config = create_mock_vllm_config(
        hf_config,
        data_parallel_size=2,
    # ... excerpt omitted for brevity ...
    assert metrics.ffn_ep_size == 8
    assert metrics.ffn_tp_size == 1
        assert per_gpu_weight_reads < global_weight_reads
        # Global should read more experts than per-gpu
        # Exact ratio depends on num_activated_experts calculation
        ratio = global_weight_reads / per_gpu_weight_reads
        # Should be > 1 since global has more experts to read
        assert ratio > 1
```
**EN:** Test case covering `moe per GPU with expert parallelism`. It exercises `Qwen3MoeConfig, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_read_bytes_breakdown`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `moe per gpu with expert parallelism` 的测试用例。 该测试会调用 `Qwen3MoeConfig, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_read_bytes_breakdown`。 代码主体包含 4 个显式断言。

### test_moe_per_gpu_expert_activation_accounting (lines 757-809)
```python
def test_moe_per_gpu_expert_activation_accounting():
    """
    Test that MoE correctly accounts for expert activations with small batch sizes.
    hf_config = Qwen3MoeConfig(
        hidden_size=2048,
        intermediate_size=8192,
        num_hidden_layers=12,
        num_experts=64,
        num_experts_per_tok=8,
        moe_intermediate_size=14336,
        n_shared_experts=0,  # No shared experts for this test
    )

    # Test with EP=8
    vllm_config = create_mock_vllm_config(
        hf_config,
        data_parallel_size=8,
    # ... excerpt omitted for brevity ...
        assert small_weight == large_weight
        # But input activation reads should scale with T*E
        small_input = small_read["routed_up_gate_input"]
        large_input = large_read["routed_up_gate_input"]
        assert large_input == 100 * small_input  # 1000/10 = 100x
```
**EN:** Test case covering `moe per GPU expert activation accounting`. It exercises `Qwen3MoeConfig, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_read_bytes_breakdown`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `moe per gpu expert activation accounting` 的测试用例。 该测试会调用 `Qwen3MoeConfig, create_mock_vllm_config, FfnMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_read_bytes_breakdown`。 代码主体包含 2 个显式断言。

### test_unembed_per_gpu_with_tensor_parallelism (lines 812-842)
```python
def test_unembed_per_gpu_with_tensor_parallelism():
    """Test unembed metrics with tensor parallelism - per_gpu vs global."""
    hf_config = Qwen3Config(
        hidden_size=4096,
        vocab_size=128000,
    )

    # Test with TP=8
    vllm_config = create_mock_vllm_config(hf_config, tensor_parallel_size=8)
    metrics = UnembedMetrics.from_vllm_config(vllm_config)

    ctx = ExecutionContext.from_single_request(
        num_tokens=100, context_len=512, is_prefill=True
    )

    # Get global and per-gpu metrics
    global_flops = metrics.get_num_flops(ctx, per_gpu=False)
    per_gpu_flops = metrics.get_num_flops(ctx, per_gpu=True)

    # With TP=8, vocab is divided by 8, so global should be 8x per-gpu
    assert global_flops == 8 * per_gpu_flops

    # For read bytes, weight reads scale with TP but input reads don't (replicated)
    global_read_breakdown = metrics.get_read_bytes_breakdown(ctx, per_gpu=False)
    per_gpu_read_breakdown = metrics.get_read_bytes_breakdown(ctx, per_gpu=True)

    # Input reads should be the same (replicated across TP ranks)
    assert global_read_breakdown["input"] == per_gpu_read_breakdown["input"]

    # Weight reads should scale 8x (divided by TP)
    assert global_read_breakdown["weight"] == 8 * per_gpu_read_breakdown["weight"]
```
**EN:** Test case covering `unembed per GPU with tensor parallelism`. It exercises `Qwen3Config, create_mock_vllm_config, UnembedMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops, metrics.get_read_bytes_breakdown`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `unembed per gpu with tensor parallelism` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, UnembedMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops, metrics.get_read_bytes_breakdown`。 代码主体包含 3 个显式断言。

### test_model_metrics_per_gpu_aggregation (lines 845-884)
```python
def test_model_metrics_per_gpu_aggregation():
    """Test ModelMetrics correctly aggregates per_gpu metrics across components."""
    hf_config = Qwen3Config(
        hidden_size=2048,
        num_attention_heads=16,
        num_hidden_layers=12,
        vocab_size=32000,
        intermediate_size=8192,
    )

    # Test with mixed parallelism: TP=2, PP=2
    vllm_config = create_mock_vllm_config(
        hf_config,
        tensor_parallel_size=2,
        pipeline_parallel_size=2,
    model_metrics = ModelMetrics(vllm_config)
    # ... excerpt omitted for brevity ...
    assert per_gpu_total == sum(per_gpu_breakdown.values())
    assert global_total == sum(global_breakdown.values())
    assert global_total > per_gpu_total
    # With TP=2 and PP=2, the ratio depends on which parallelism applies to
    # which component but we can verify that global is reasonably larger
    ratio = global_total / per_gpu_total
    assert ratio > 1  # Should be between PP and TP*PP depending on component mix
```
**EN:** Test case covering `model metrics per GPU aggregation`. It exercises `Qwen3Config, create_mock_vllm_config, ModelMetrics, ExecutionContext.from_single_request, model_metrics.get_num_flops_breakdown, model_metrics.get_num_flops`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `model metrics per gpu aggregation` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, ModelMetrics, ExecutionContext.from_single_request, model_metrics.get_num_flops_breakdown, model_metrics.get_num_flops`。 代码主体包含 4 个显式断言。

### test_attention_per_gpu_heads_not_evenly_divisible (lines 887-910)
```python
def test_attention_per_gpu_heads_not_evenly_divisible():
    """Test attention with heads not evenly divisible by TP."""
    hf_config = Qwen3Config(
        hidden_size=2048,
        num_attention_heads=17,  # Not divisible by 4
        num_key_value_heads=5,  # Not divisible by 4
        num_hidden_layers=8,
    )

    vllm_config = create_mock_vllm_config(hf_config, tensor_parallel_size=4)
    metrics = AttentionMetrics.from_vllm_config(vllm_config)

    ctx = ExecutionContext.from_single_request(
        num_tokens=64, context_len=256, is_prefill=True
    )

    # Should not crash and should handle max(1, ...) correctly
    per_gpu_flops = metrics.get_num_flops(ctx, per_gpu=True)
    global_flops = metrics.get_num_flops(ctx, per_gpu=False)

    # Both should be positive
    assert per_gpu_flops > 0
    assert global_flops > 0
    assert global_flops > per_gpu_flops
```
**EN:** Test case covering `attention per GPU heads not evenly divisible`. It exercises `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `注意力 per gpu heads not evenly divisible` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, AttentionMetrics.from_vllm_config, ExecutionContext.from_single_request, metrics.get_num_flops`。 代码主体包含 3 个显式断言。

### Module state / 模块级状态 (line 914)
```python
_INT4_FP4_METHODS = [m for m, s in _QUANT_WEIGHT_BYTE_SIZE.items() if s == 0.5]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_INT4_FP4_METHODS`. Shared setup calls include `_QUANT_WEIGHT_BYTE_SIZE.items`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_INT4_FP4_METHODS`。 共享初始化调用包括 `_QUANT_WEIGHT_BYTE_SIZE.items`。

### test_quantization_config_parser_int4_methods (lines 918-941)
```python
def test_quantization_config_parser_int4_methods(quant_method):
    """Test quantization parsers with INT4/FP4 methods (0.5 bytes)."""

    class MockQuantConfig:
        def get_name(self):
            return quant_method

    hf_config = Qwen3Config(
        hidden_size=2048,
        num_attention_heads=16,
        intermediate_size=8192,
        num_hidden_layers=1,
    )
    vllm_config = create_mock_vllm_config(hf_config, quant_config=MockQuantConfig())

    attn_result = AttentionMetrics.get_parser().parse(vllm_config)
    assert attn_result.weight_byte_size == 0.5, (
        f"Expected 0.5 for {quant_method}, got {attn_result.weight_byte_size}"
    )

    ffn_result = FfnMetrics.get_parser().parse(vllm_config)
    assert ffn_result.weight_byte_size == 0.5, (
        f"Expected 0.5 for {quant_method}, got {ffn_result.weight_byte_size}"
    )
```
**EN:** Parameterized test covering `quantization config parser int4 methods`. Parameter axes: `quant_method`. Inputs/fixtures: `quant_method`. It exercises `mark.parametrize, Qwen3Config, create_mock_vllm_config, get_parser.parse, MockQuantConfig, AttentionMetrics.get_parser`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `quantization config parser int4 methods` 的测试用例。 参数维度：`quant_method`。 输入或 fixture：`quant_method`。 该测试会调用 `mark.parametrize, Qwen3Config, create_mock_vllm_config, get_parser.parse, MockQuantConfig, AttentionMetrics.get_parser`。 代码主体包含 2 个显式断言。

### Module state / 模块级状态 (line 945)
```python
_FP8_INT8_METHODS = [m for m, s in _QUANT_WEIGHT_BYTE_SIZE.items() if s == 1]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_FP8_INT8_METHODS`. Shared setup calls include `_QUANT_WEIGHT_BYTE_SIZE.items`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_FP8_INT8_METHODS`。 共享初始化调用包括 `_QUANT_WEIGHT_BYTE_SIZE.items`。

### test_quantization_config_parser_fp8_methods (lines 949-972)
```python
def test_quantization_config_parser_fp8_methods(quant_method):
    """Test quantization parsers with FP8/INT8 methods (1 byte)."""

    class MockQuantConfig:
        def get_name(self):
            return quant_method

    hf_config = Qwen3Config(
        hidden_size=2048,
        num_attention_heads=16,
        intermediate_size=8192,
        num_hidden_layers=1,
    )
    vllm_config = create_mock_vllm_config(hf_config, quant_config=MockQuantConfig())

    attn_result = AttentionMetrics.get_parser().parse(vllm_config)
    assert attn_result.weight_byte_size == 1, (
        f"Expected 1 for {quant_method}, got {attn_result.weight_byte_size}"
    )

    ffn_result = FfnMetrics.get_parser().parse(vllm_config)
    assert ffn_result.weight_byte_size == 1, (
        f"Expected 1 for {quant_method}, got {ffn_result.weight_byte_size}"
    )
```
**EN:** Parameterized test covering `quantization config parser fp8 methods`. Parameter axes: `quant_method`. Inputs/fixtures: `quant_method`. It exercises `mark.parametrize, Qwen3Config, create_mock_vllm_config, get_parser.parse, MockQuantConfig, AttentionMetrics.get_parser`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `quantization config parser fp8 methods` 的测试用例。 参数维度：`quant_method`。 输入或 fixture：`quant_method`。 该测试会调用 `mark.parametrize, Qwen3Config, create_mock_vllm_config, get_parser.parse, MockQuantConfig, AttentionMetrics.get_parser`。 代码主体包含 2 个显式断言。

### test_quantization_config_parser_unknown_method (lines 975-994)
```python
def test_quantization_config_parser_unknown_method():
    """Test that an unrecognized quant method raises InvalidComponent."""

    class MockQuantConfig:
        def get_name(self):
            return "unknown_quant_method"

    hf_config = Qwen3Config(
        hidden_size=2048,
        num_attention_heads=16,
        intermediate_size=8192,
        num_hidden_layers=1,
    )
    vllm_config = create_mock_vllm_config(hf_config, quant_config=MockQuantConfig())

    with pytest.raises(InvalidComponent):
        AttentionMetrics.get_parser().parse(vllm_config)

    with pytest.raises(InvalidComponent):
        FfnMetrics.get_parser().parse(vllm_config)
```
**EN:** Test case covering `quantization config parser unknown method`. It exercises `Qwen3Config, create_mock_vllm_config, pytest.raises, get_parser.parse, MockQuantConfig, AttentionMetrics.get_parser`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `quantization config parser unknown method` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, pytest.raises, get_parser.parse, MockQuantConfig, AttentionMetrics.get_parser`。 主要通过预期异常检查来完成验证。

### test_quantized_model_metrics_aggregation (lines 997-1023)
```python
def test_quantized_model_metrics_aggregation():
    """Test that ModelMetrics works end-to-end with a quantized model config."""

    class MockQuantConfig:
        def get_name(self):
            return "gptq"

    hf_config = Qwen3Config(
        hidden_size=2048,
        num_attention_heads=16,
        num_hidden_layers=12,
        vocab_size=32000,
        intermediate_size=8192,
    )
    vllm_config = create_mock_vllm_config(hf_config, quant_config=MockQuantConfig())

    model_metrics = ModelMetrics(vllm_config)
    ctx = ExecutionContext.from_single_request(
        num_tokens=100, context_len=512, is_prefill=True
    )

    # Should not crash and should produce valid metrics
    total_flops = model_metrics.get_num_flops(ctx)
    breakdown = model_metrics.get_num_flops_breakdown(ctx)

    assert total_flops > 0
    assert total_flops == sum(breakdown.values())
```
**EN:** Test case covering `quantized model metrics aggregation`. It exercises `Qwen3Config, create_mock_vllm_config, ModelMetrics, ExecutionContext.from_single_request, model_metrics.get_num_flops, model_metrics.get_num_flops_breakdown`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `quantized model metrics aggregation` 的测试用例。 该测试会调用 `Qwen3Config, create_mock_vllm_config, ModelMetrics, ExecutionContext.from_single_request, model_metrics.get_num_flops, model_metrics.get_num_flops_breakdown`。 代码主体包含 2 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, transformers.models.deepseek_v3.configuration_deepseek_v3, transformers.models.llama4.configuration_llama4, transformers.models.qwen3.configuration_qwen3, transformers.models.qwen3_moe.configuration_qwen3_moe`.
- **CN:** 外部库：`pytest, transformers.models.deepseek_v3.configuration_deepseek_v3, transformers.models.llama4.configuration_llama4, transformers.models.qwen3.configuration_qwen3, transformers.models.qwen3_moe.configuration_qwen3_moe`。
- **EN:** vLLM modules under test: `vllm.config.model, vllm.transformers_utils.model_arch_config_convertor, vllm.v1.metrics.perf`.
- **CN:** 被测试的 vLLM 模块：`vllm.config.model, vllm.transformers_utils.model_arch_config_convertor, vllm.v1.metrics.perf`。
- **EN:** Standard-library support: `types`.
- **CN:** 标准库支持：`types`。

# kt_ep_wrapper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/kt_ep_wrapper.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `KTConfig`, `create_kt_config_from_server_args`, `mask_cpu_expert_ids`, and `KTEPWrapperMethod` and connects them to backend-specific paths such as `CUDA`, `NPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `KTConfig`、`create_kt_config_from_server_args`、`mask_cpu_expert_ids` 以及 `KTEPWrapperMethod` 等符号，并把这些符号连接到 `CUDA`、`NPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and module overview
```python
# SPDX-License-Identifier: Apache-2.0
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 2-34: Imports, conditional backend setup, and runtime guards
```python
"""
KT Expert Parallelism Wrapper for MoE layers.

This module provides a generic wrapper that enables CPU-GPU expert parallelism
for any MoE quantization method. It coordinates parallel execution of GPU experts
(using any quantization method) and CPU experts (using AMX/AVX instructions).
"""

from dataclasses import dataclass
from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.distributed import get_tensor_model_parallel_rank
from sglang.srt.layers.quantization.base_config import FusedMoEMethodBase
from sglang.srt.utils import get_compiler_backend

if TYPE_CHECKING:
    from sglang.srt.layers.moe import MoeRunnerConfig
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
    from sglang.srt.server_args import ServerArgs

try:
    from kt_kernel import KTMoEWrapper

    KTRANSFORMERS_AVAILABLE = True
except ImportError:
    KTRANSFORMERS_AVAILABLE = False
```
**EN:** This section prepares the module namespace. It imports `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.Optional`, `torch`, `sglang.srt.distributed.get_tensor_model_parallel_rank`, and `sglang.srt.layers.quantization.base_config.FusedMoEMethodBase`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `KTRANSFORMERS_AVAILABLE` capture configuration, cached handles, or feature flags. Control structures like `If` and `Try` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.Optional`、`torch`、`sglang.srt.distributed.get_tensor_model_parallel_rank` 以及 `sglang.srt.layers.quantization.base_config.FusedMoEMethodBase`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `KTRANSFORMERS_AVAILABLE` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 和 `Try` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 35-60: Class `KTConfig` declaration and shared state
```python
@dataclass
class KTConfig:
    """Configuration for KTransformers heterogeneous computing CPU part.

    Args:
        layer_idx: Layer index in the model
        num_gpu_experts: Number of experts to run on GPU
        cpuinfer_threads: Number of CPU inference threads
        threadpool_count: Number of thread pools for CPU computation
        weight_path: Path to CPU quantized weights
        chunked_prefill_size: Chunk size for prefill computation
        method: CPU computation method (e.g., "int4")
        num_layers: Total number of layers in the model (optional)
    """

    layer_idx: int
    num_gpu_experts: int
    cpuinfer_threads: int
    threadpool_count: int
    weight_path: str
    chunked_prefill_size: int
    max_deferred_experts_per_token: int
    method: str
    num_layers: Optional[int] = None
```
**EN:** This block introduces class `KTConfig` and the state shared by its methods. The class docstring summarizes its role: Configuration for KTransformers heterogeneous computing CPU part.
**CN:** 该代码块引入类 `KTConfig`，并定义其方法共享的状态。 类级文档进一步概括了它的职责。

### Lines 61-97: `create_kt_config_from_server_args` constructor for kt config from server args
```python
def create_kt_config_from_server_args(
    server_args: "ServerArgs", layer_idx: int
) -> Optional[KTConfig]:
    """Create KTConfig from ServerArgs if KT is configured.

    Args:
        server_args: Global server arguments
        layer_idx: Layer index in the model

    Returns:
        KTConfig if KT is configured, None otherwise
    """
    if server_args.kt_weight_path is None:
        return None

    # Try to get num_layers from model config
    num_layers = None
    try:
        hf_config = server_args.get_hf_config()
        num_layers = getattr(hf_config, "num_hidden_layers", None)
    except Exception:
        # If we can't get the config, num_layers will be None
        pass

    return KTConfig(
        layer_idx=layer_idx,
        num_gpu_experts=server_args.kt_num_gpu_experts,
        cpuinfer_threads=server_args.kt_cpuinfer,
        threadpool_count=server_args.kt_threadpool_count,
        weight_path=server_args.kt_weight_path,
        chunked_prefill_size=server_args.chunked_prefill_size,
        method=server_args.kt_method,
        max_deferred_experts_per_token=server_args.kt_max_deferred_experts_per_token,
        num_layers=num_layers,
    )
```
**EN:** This block defines `create_kt_config_from_server_args` and contains the main logic for this step. It mainly invokes `KTConfig`, `server_args.get_hf_config`, and `getattr`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_layers` and `hf_config` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `create_kt_config_from_server_args`，并承载这一阶段的核心逻辑。 它主要调用 `KTConfig`、`server_args.get_hf_config` 以及 `getattr`，说明该流程会编排底层辅助函数或计算内核。 像 `num_layers` 和 `hf_config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 98-116: Function `mask_cpu_expert_ids` and its core logic
```python
@torch.compile(dynamic=True, backend=get_compiler_backend())
def mask_cpu_expert_ids(topk_ids: torch.Tensor, num_gpu_experts: int) -> torch.Tensor:
    """Mask CPU expert IDs by setting them to -1.

    This function masks expert IDs that should be computed on CPU (IDs >= num_gpu_experts)
    so they won't be computed on GPU. The masked IDs are set to -1, which causes the
    GPU MoE kernel to skip those experts.

    Args:
        topk_ids: Tensor of shape [num_tokens, top_k] containing expert IDs
        num_gpu_experts: Number of experts that should run on GPU (experts 0 to num_gpu_experts-1)

    Returns:
        Modified topk_ids tensor with CPU expert IDs masked as -1
    """
    topk_ids[topk_ids >= num_gpu_experts] = -1
    return topk_ids
```
**EN:** This block defines `mask_cpu_expert_ids` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.compile` and `get_compiler_backend`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `mask_cpu_expert_ids`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.compile` 和 `get_compiler_backend`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 117-135: Class `KTEPWrapperMethod` declaration and shared state
```python
class KTEPWrapperMethod(FusedMoEMethodBase):
    """Wrapper for any MoE quantization method to enable CPU-GPU expert parallelism.

    This wrapper coordinates parallel execution of:
    - GPU experts (0 to num_gpu_experts-1) using any quantization method
    - CPU experts (num_gpu_experts to total_experts-1) using AMX/AVX instructions

    The wrapper implements the submit-compute-sync pattern:
    1. Submit CPU expert computation (non-blocking)
    2. Execute GPU expert computation in parallel
    3. Synchronize and merge CPU+GPU results

    Example:
        # Wrap any GPU method with AMX/AVX CPU expert support
        gpu_method = CompressedTensorsWNA16MoE(quant_config, prefix)
        kt_config = KTConfig(layer_idx=0, num_gpu_experts=4, ...)
        method = KTEPWrapperMethod(gpu_method, kt_config)
    """
```
**EN:** This block introduces class `KTEPWrapperMethod` and the state shared by its methods. It inherits from `FusedMoEMethodBase`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Wrapper for any MoE quantization method to enable CPU-GPU expert parallelism.
**CN:** 该代码块引入类 `KTEPWrapperMethod`，并定义其方法共享的状态。 它继承自 `FusedMoEMethodBase`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 136-164: `KTEPWrapperMethod` initialization and state setup
```python
    def __init__(
        self,
        gpu_method: FusedMoEMethodBase,
        kt_config: KTConfig,
    ):
        """Initialize the KT EP wrapper.

        Args:
            gpu_method: The quantization method to use for GPU experts
            kt_config: Configuration for KT CPU expert computation
        """
        if not KTRANSFORMERS_AVAILABLE:
            raise ImportError(
                "kt_kernel is not installed. To use KTransformers EP wrapper, please install kt_kernel."
            )

        self.gpu_method = gpu_method
        self.kt_config = kt_config
        self.num_gpu_experts = kt_config.num_gpu_experts
        self.override_num_local_experts = True
        self.gpu_method.num_gpu_experts = self.num_gpu_experts
        self.tp_rank = get_tensor_model_parallel_rank()

        # KT wrapper will be initialized in create_weights
        self.wrapper: Optional[KTMoEWrapper] = None

        # Store parameters needed for KT initialization
        self._layer_params = None
```
**EN:** This block defines `KTEPWrapperMethod.__init__` and contains the main logic for this step. It mainly invokes `get_tensor_model_parallel_rank` and `ImportError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.gpu_method`, `self.kt_config`, `self.num_gpu_experts`, `self.override_num_local_experts`, and `self.tp_rank` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `KTEPWrapperMethod.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `get_tensor_model_parallel_rank` 和 `ImportError`，说明该流程会编排底层辅助函数或计算内核。 像 `self.gpu_method`、`self.kt_config`、`self.num_gpu_experts`、`self.override_num_local_experts` 以及 `self.tp_rank` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 165-233: `KTEPWrapperMethod.create_weights` constructor for weights
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        """Create weights for both GPU and CPU experts.

        Args:
            layer: The MoE layer module
            num_experts: Total number of experts (GPU + CPU)
            hidden_size: Hidden dimension size
            intermediate_size_per_partition: Intermediate size per TP partition
            params_dtype: Data type for parameters
            **extra_weight_attrs: Additional weight attributes
        """
        self.global_num_experts = num_experts
        self.hidden_size = hidden_size
        self.intermediate_size_per_partition = intermediate_size_per_partition

        # Get required parameters from layer object
        # top_k: number of experts selected per token
        num_experts_per_tok = layer.top_k

        # intermediate_size_full: full intermediate size before TP partitioning
        intermediate_size_full = (
            layer.intermediate_size_per_partition * layer.moe_tp_size
        )

        layer_max_deferred = self.kt_config.max_deferred_experts_per_token or 0
        if (
            self.kt_config.max_deferred_experts_per_token is not None
            and self.kt_config.num_layers is not None
            and self.kt_config.layer_idx == self.kt_config.num_layers - 1
        ):
            layer_max_deferred = 0

        # 1. Create weights for GPU experts using the wrapped method
        # GPU experts: 0 to num_gpu_experts-1
        self.gpu_method.create_weights(
            layer=layer,
            num_experts=self.num_gpu_experts,
            hidden_size=hidden_size,
            intermediate_size_per_partition=intermediate_size_per_partition,
            params_dtype=params_dtype,
            **extra_weight_attrs,
        )

        # 2. Initialize KT wrapper for CPU experts
        # CPU experts: num_gpu_experts to num_experts-1
        if self.tp_rank == 0:
            self.wrapper = KTMoEWrapper(
                layer_idx=self.kt_config.layer_idx,
                num_experts=num_experts,
                num_experts_per_tok=num_experts_per_tok,
                hidden_size=hidden_size,
                moe_intermediate_size=intermediate_size_full,
                num_gpu_experts=self.num_gpu_experts,
                cpuinfer_threads=self.kt_config.cpuinfer_threads,
                threadpool_count=self.kt_config.threadpool_count,
                weight_path=self.kt_config.weight_path,
                chunked_prefill_size=self.kt_config.chunked_prefill_size,
                method=self.kt_config.method,
                max_deferred_experts_per_token=layer_max_deferred,
            )
```
**EN:** This block defines `KTEPWrapperMethod.create_weights` and contains the main logic for this step. It mainly invokes `self.gpu_method.create_weights` and `KTMoEWrapper`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.global_num_experts`, `self.hidden_size`, `self.intermediate_size_per_partition`, `num_experts_per_tok`, and `intermediate_size_full` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `KTEPWrapperMethod.create_weights`，并承载这一阶段的核心逻辑。 它主要调用 `self.gpu_method.create_weights` 和 `KTMoEWrapper`，说明该流程会编排底层辅助函数或计算内核。 像 `self.global_num_experts`、`self.hidden_size`、`self.intermediate_size_per_partition`、`num_experts_per_tok` 以及 `intermediate_size_full` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 234-259: Function `KTEPWrapperMethod.process_weights_after_loading` and its core logic
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        """Process weights after loading from checkpoint.

        Args:
            layer: The MoE layer module
        """
        # 1. Process GPU weights
        if hasattr(self.gpu_method, "process_weights_after_loading"):
            self.gpu_method.process_weights_after_loading(layer)

        # 2. Load CPU weights using KT wrapper
        if self.tp_rank == 0 and self.wrapper is not None:
            torch.cuda.synchronize()

            # Get expert location metadata for CPU expert mapping
            from sglang.srt.eplb.expert_location_dispatch import (
                get_global_expert_location_metadata,
            )

            physical_to_logical_map_cpu = (
                get_global_expert_location_metadata()
                .physical_to_logical_map_cpu[self.kt_config.layer_idx]
                .contiguous()
            )
            self.wrapper.load_weights(physical_to_logical_map_cpu)
```
**EN:** This block defines `KTEPWrapperMethod.process_weights_after_loading` and contains the main logic for this step. It mainly invokes `hasattr`, `self.gpu_method.process_weights_after_loading`, `torch.cuda.synchronize`, `get_global_expert_location_metadata.physical_to_logical_map_cpu.contiguous`, and `self.wrapper.load_weights`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `physical_to_logical_map_cpu` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `KTEPWrapperMethod.process_weights_after_loading`，并承载这一阶段的核心逻辑。 它主要调用 `hasattr`、`self.gpu_method.process_weights_after_loading`、`torch.cuda.synchronize`、`get_global_expert_location_metadata.physical_to_logical_map_cpu.contiguous` 以及 `self.wrapper.load_weights`，说明该流程会编排底层辅助函数或计算内核。 像 `physical_to_logical_map_cpu` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 260-274: `KTEPWrapperMethod.create_moe_runner` constructor for Mixture-of-Experts runner
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: "MoeRunnerConfig"
    ):
        """Create MoE runner for computation.

        Args:
            layer: The MoE layer module
            moe_runner_config: Configuration for MoE runner
        """
        self.moe_runner_config = moe_runner_config
        if self.override_num_local_experts:
            moe_runner_config.num_local_experts = self.num_gpu_experts
        # Delegate to GPU method to create its runner
        self.gpu_method.create_moe_runner(layer, moe_runner_config)
```
**EN:** This block defines `KTEPWrapperMethod.create_moe_runner` and contains the main logic for this step. It mainly invokes `self.gpu_method.create_moe_runner`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.moe_runner_config` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `KTEPWrapperMethod.create_moe_runner`，并承载这一阶段的核心逻辑。 它主要调用 `self.gpu_method.create_moe_runner`，说明该流程会编排底层辅助函数或计算内核。 像 `self.moe_runner_config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 275-304: Function `KTEPWrapperMethod.submit` and its core logic
```python
    def submit(
        self,
        layer: torch.nn.Module,
        dispatch_output: "StandardDispatchOutput",
    ) -> None:
        """Submit CPU expert computation asynchronously (non-blocking).

        This method submits the CPU expert computation to AMX/AVX without waiting
        for completion, allowing GPU computation to proceed in parallel.

        Args:
            layer: The MoE layer module
            dispatch_output: Dispatched tokens and routing information
        """
        assert (
            self.moe_runner_config.activation == "silu"
        ), "Only SiLU activation is supported."

        if self.tp_rank != 0 or self.wrapper is None:
            return

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output
        topk_weights, topk_ids, _ = topk_output

        # Submit forward task to CPU (non-blocking)
        self.wrapper.submit_forward(
            x, topk_ids, topk_weights, torch.cuda.current_stream(x.device).cuda_stream
        )
```
**EN:** This block defines `KTEPWrapperMethod.submit` and contains the main logic for this step. It mainly invokes `self.wrapper.submit_forward` and `torch.cuda.current_stream`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x`, `topk_output`, `topk_weights`, `topk_ids`, and `_` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `KTEPWrapperMethod.submit`，并承载这一阶段的核心逻辑。 它主要调用 `self.wrapper.submit_forward` 和 `torch.cuda.current_stream`，说明该流程会编排底层辅助函数或计算内核。 像 `x`、`topk_output`、`topk_weights`、`topk_ids` 以及 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 305-323: Function `KTEPWrapperMethod.sync` and its core logic
```python
    def sync(self, x: torch.Tensor) -> torch.Tensor:
        """Synchronize and retrieve CPU expert computation results.

        This method waits for the CPU computation to complete and returns the results.

        Args:
            x: Reference tensor for shape and device information

        Returns:
            CPU expert computation results
        """
        if self.tp_rank != 0 or self.wrapper is None:
            return torch.zeros_like(x)

        # Wait for CPU computation and retrieve results
        return self.wrapper.sync_forward(
            x, torch.cuda.current_stream(x.device).cuda_stream
        )
```
**EN:** This block defines `KTEPWrapperMethod.sync` and contains the main logic for this step. It mainly invokes `self.wrapper.sync_forward`, `torch.zeros_like`, and `torch.cuda.current_stream`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `KTEPWrapperMethod.sync`，并承载这一阶段的核心逻辑。 它主要调用 `self.wrapper.sync_forward`、`torch.zeros_like` 以及 `torch.cuda.current_stream`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 324-374: Function `KTEPWrapperMethod.apply` and its core logic
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: "StandardDispatchOutput",
    ) -> "CombineInput":
        """Execute hybrid CPU+GPU MoE forward pass with parallelism.

        This is the main computation method that coordinates:
        1. Submit CPU expert computation (non-blocking)
        2. Execute GPU expert computation in parallel
        3. Synchronize CPU results and merge with GPU results

        Args:
            layer: The MoE layer module
            dispatch_output: Dispatched tokens and routing information

        Returns:
            Combined computation results from CPU and GPU experts
        """
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        # Step 1: Submit CPU expert computation (non-blocking)
        if self.tp_rank == 0:
            self.submit(layer, dispatch_output)

        # Step 2: Prepare GPU computation by masking CPU expert IDs
        # CPU expert IDs (>= num_gpu_experts) are set to -1 so GPU kernel skips them
        topk_ids = topk_output.topk_ids
        masked_topk_ids = mask_cpu_expert_ids(topk_ids, self.num_gpu_experts)

        # Create modified dispatch output for GPU computation
        masked_topk_output = topk_output._replace(topk_ids=masked_topk_ids)
        masked_dispatch_output = dispatch_output._replace(
            topk_output=masked_topk_output
        )

        # Step 3: Execute GPU expert computation (any quantization method)
        # This runs in parallel with CPU computation
        gpu_combine_input = self.gpu_method.apply(layer, masked_dispatch_output)

        # Step 4: Synchronize CPU results and merge with GPU results
        output = gpu_combine_input.hidden_states
        if self.tp_rank == 0:
            cpu_output = self.sync(x)
            output = output + cpu_output

        return StandardCombineInput(hidden_states=output)
```
**EN:** This block defines `KTEPWrapperMethod.apply` and contains the main logic for this step. It mainly invokes `mask_cpu_expert_ids`, `topk_output._replace`, `dispatch_output._replace`, `self.gpu_method.apply`, and `StandardCombineInput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x`, `topk_output`, `topk_ids`, `masked_topk_ids`, and `masked_topk_output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `KTEPWrapperMethod.apply`，并承载这一阶段的核心逻辑。 它主要调用 `mask_cpu_expert_ids`、`topk_output._replace`、`dispatch_output._replace`、`self.gpu_method.apply` 以及 `StandardCombineInput`，说明该流程会编排底层辅助函数或计算内核。 像 `x`、`topk_output`、`topk_ids`、`masked_topk_ids` 以及 `masked_topk_output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 375-393: Internal helper `KTEPWrapperMethod.__getattr__`
```python
    def __getattr__(self, name: str):
        """Delegate attribute access to the wrapped GPU method.

        This allows the wrapper to transparently expose attributes and methods
        from the wrapped GPU quantization method.

        Args:
            name: Attribute name

        Returns:
            Attribute value from gpu_method
        """
        # Avoid infinite recursion for internal attributes
        if name in ("gpu_method", "wrapper", "kt_config"):
            raise AttributeError(
                f"'{type(self).__name__}' object has no attribute '{name}'"
            )

        return getattr(self.gpu_method, name)
```
**EN:** This block defines `KTEPWrapperMethod.__getattr__` and contains the main logic for this step. It mainly invokes `getattr`, `AttributeError`, and `type`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `KTEPWrapperMethod.__getattr__`，并承载这一阶段的核心逻辑。 它主要调用 `getattr`、`AttributeError` 以及 `type`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `KTConfig`, `create_kt_config_from_server_args`, `mask_cpu_expert_ids`, and `KTEPWrapperMethod`. / **主要符号**：核心入口包括 `KTConfig`、`create_kt_config_from_server_args`、`mask_cpu_expert_ids` 以及 `KTEPWrapperMethod`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `dataclasses.dataclass`, `typing.TYPE_CHECKING`, and `typing.Optional` / **标准库**：`dataclasses.dataclass`、`typing.TYPE_CHECKING` 以及 `typing.Optional`
- **Third-party**: `torch` and `kt_kernel.KTMoEWrapper` / **第三方依赖**：`torch` 和 `kt_kernel.KTMoEWrapper`
- **Internal SGLang modules**: `sglang.srt.distributed.get_tensor_model_parallel_rank`, `sglang.srt.layers.quantization.base_config.FusedMoEMethodBase`, `sglang.srt.utils.get_compiler_backend`, `sglang.srt.layers.moe.MoeRunnerConfig`, `sglang.srt.layers.moe.token_dispatcher.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.StandardDispatchOutput`, `sglang.srt.server_args.ServerArgs`, `sglang.srt.layers.moe.token_dispatcher.StandardCombineInput`, and `sglang.srt.eplb.expert_location_dispatch.get_global_expert_location_metadata` / **SGLang 内部模块**：`sglang.srt.distributed.get_tensor_model_parallel_rank`、`sglang.srt.layers.quantization.base_config.FusedMoEMethodBase`、`sglang.srt.utils.get_compiler_backend`、`sglang.srt.layers.moe.MoeRunnerConfig`、`sglang.srt.layers.moe.token_dispatcher.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.StandardDispatchOutput`、`sglang.srt.server_args.ServerArgs`、`sglang.srt.layers.moe.token_dispatcher.StandardCombineInput` 以及 `sglang.srt.eplb.expert_location_dispatch.get_global_expert_location_metadata`

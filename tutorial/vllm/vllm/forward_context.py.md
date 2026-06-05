# forward_context.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `vllm/forward_context.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Carries per-forward-pass execution metadata such as attention state, DP/SP sizing, CUDA-graph mode, and timing statistics. / 保存单次前向执行所需的上下文信息，例如注意力元数据、DP/SP 尺寸、CUDA graph 模式与耗时统计。

## Line-by-Line Analysis / 逐行分析
### Batch tracking globals and batch descriptor (lines 22-60)
```python
track_batchsize: bool = envs.VLLM_LOG_BATCHSIZE_INTERVAL >= 0
last_logging_time: float = 0
forward_start_time: float = 0
batchsize_logging_interval: float = envs.VLLM_LOG_BATCHSIZE_INTERVAL
batchsize_forward_time: defaultdict = defaultdict(list)


@dataclass(frozen=True)
class BatchDescriptor:
    """
    Batch descriptor for cudagraph dispatching. We should keep the num of
    items as minimal as possible to properly and uniquely describe the padded
    batch for cudagraph.
    """

    num_tokens: int
    num_reqs: int | None = None
    """
    Number of requests in the batch. Can be None for PIECEWISE cudagraphs where
    the cudagraphs can handle any number of requests.
    """
    uniform: bool = False
    """
    True if all the requests in the batch have the same number of tokens.
    """
    has_lora: bool = False
    """
    Whether this batch has active LoRA adapters.
    """
    num_active_loras: int = 0
    """
    Number of distinct active LoRA adapters in this batch.
    When cudagraph_specialize_lora_count is enabled, separate CUDA graphs
    are captured for each num_active_loras value. This allows kernels
    (like fused_moe_lora) whose grid size depends on num_active_loras
    to be properly captured.
    """


```
**EN:** The module-level flags (`track_batchsize`, `batchsize_forward_time`, etc.) implement optional telemetry controlled by `envs.VLLM_LOG_BATCHSIZE_INTERVAL`. `BatchDescriptor` is the compact shape signature used by CUDA-graph dispatch: number of tokens, optional request count, uniformity, and LoRA cardinality are the minimum facts needed to choose a captured graph safely.
**CN:** 模块级变量（如 `track_batchsize`、`batchsize_forward_time` 等）实现了可选遥测功能，其开关由 `envs.VLLM_LOG_BATCHSIZE_INTERVAL` 控制。`BatchDescriptor` 则是 CUDA graph 分发使用的紧凑批次签名：token 数、可选请求数、是否均匀以及 LoRA 数量，正好构成安全选择已捕获图所需的最小信息集。

### Sequence/data-parallel sizing helpers (lines 61-126)
```python
def _compute_sp_num_tokens(
    num_tokens_across_dp_cpu: torch.Tensor, sequence_parallel_size: int
) -> list[int]:
    sp_tokens = (
        num_tokens_across_dp_cpu + sequence_parallel_size - 1
    ) // sequence_parallel_size

    sp_tokens = sp_tokens.repeat_interleave(sequence_parallel_size)
    return sp_tokens.tolist()


@dataclass
class DPMetadata:
    num_tokens_across_dp_cpu: torch.Tensor

    # NOTE: local_sizes should only be set by the chunked_sizes context manager
    local_sizes: list[int] | None = None

    @staticmethod
    def make(
        parallel_config: ParallelConfig,
        num_tokens: int,
        num_tokens_across_dp_cpu: torch.Tensor,
    ) -> "DPMetadata":
        assert num_tokens_across_dp_cpu is not None
        assert parallel_config.data_parallel_size > 1
        assert parallel_config.is_moe_model is not False
        dp_rank = parallel_config.data_parallel_rank
        batchsize = num_tokens

        # If num_tokens_across_dp is None, it will be computed by all_reduce
        # Otherwise, num_tokens_across_dp[dp_rank] should be equal to batchsize
        assert num_tokens_across_dp_cpu[dp_rank] == batchsize, (
            f"{num_tokens_across_dp_cpu[dp_rank]} {batchsize}"
        )
        return DPMetadata(num_tokens_across_dp_cpu)

    @contextmanager
    def sp_local_sizes(self, sequence_parallel_size: int):
        """
        Context manager for setting self.local_sizes. Same as self.chunked_sizes
        but without any chunking.
        """
        self.local_sizes = _compute_sp_num_tokens(
            self.num_tokens_across_dp_cpu, sequence_parallel_size
        )
        try:
            yield self.local_sizes
        finally:
            self.local_sizes = None

    def get_chunk_sizes_across_dp_rank(self) -> list[int] | None:
        assert self.local_sizes is not None
        return self.local_sizes

    # Get the cumulative tokens across sequence parallel ranks.
    # In this case the input to the MoEs will be distributed w.r.t both
    # DP and TP rank.
    # When sp_size==1, this is just the cumulative num tokens across DP.
    def cu_tokens_across_sp(self, sp_size: int) -> torch.Tensor:
        num_tokens_across_sp_cpu = (
            self.num_tokens_across_dp_cpu - 1 + sp_size
        ) // sp_size
        num_tokens_across_sp_cpu = num_tokens_across_sp_cpu.repeat_interleave(sp_size)
        return torch.cumsum(num_tokens_across_sp_cpu, dim=0)

```
**EN:** `_compute_sp_num_tokens()` rounds token counts up for sequence parallelism, and `DPMetadata` wraps the cross-DP token tensor with helper methods. The context manager `sp_local_sizes()` temporarily exposes rank-local chunk sizes, while `cu_tokens_across_sp()` computes cumulative token offsets used by MoE and mixed DP/SP layouts.
**CN:** `_compute_sp_num_tokens()` 会为 sequence parallelism 向上取整 token 数，`DPMetadata` 则把跨 DP 的 token 张量包装成带辅助方法的对象。上下文管理器 `sp_local_sizes()` 临时暴露各 rank 的本地分块尺寸，而 `cu_tokens_across_sp()` 负责计算 MoE 与 DP/SP 混合布局所需的累计 token 偏移。

### ForwardContext data model (lines 128-185)
```python
@dataclass
class ForwardContext:
    # copy from vllm_config.compilation_config.static_forward_context
    no_compile_layers: dict[str, Any]
    attn_metadata: dict[str, AttentionMetadata] | list[dict[str, AttentionMetadata]]
    slot_mapping: dict[str, torch.Tensor] | list[dict[str, torch.Tensor]]
    """
    Type Dict[str, AttentionMetadata] for v1, map from layer_name of each
    attention layer to its attention metadata
    Type List[Dict[str, AttentionMetadata]] for DBO. List of size two, one
    for each microbatch.
    Set dynamically for each forward pass
    """
    # set dynamically for each forward pass
    dp_metadata: DPMetadata | None = None
    # determine the cudagraph style at runtime to be FULL, PIECEWISE, or NONE.
    # by default NONE, no cudagraph is used.
    cudagraph_runtime_mode: CUDAGraphMode = CUDAGraphMode.NONE
    batch_descriptor: BatchDescriptor | None = None

    ubatch_slices: UBatchSlices | None = None

    # If True, bypass the compiled model call, e.g. by using .forward() directly
    skip_compiled: bool = False

    # For torch.compile cold start times, we need to avoid hard-coding
    # any strings into the graph. Right now, the vllm.moe_forward
    # and vllm.moe_forward_shared custom operators hard-code strings into
    # the graph.
    #
    # The workaround is to store a list of the strings that each of those
    # custom ops needs in the ForwardContext (all_moe_layers)
    # as well as a counter (moe_layer_index).
    # The ForwardContext object is alive for the duration of the forward pass.
    # When the custom op needs a layer string, get the next string
    # from all_moe_layers and increment the counter.
    #
    # This assumes that the custom operators will always be executed in
    # order and that torch.compile will not try to reorder these
    # operations with respect to each other.
    #
    # TODO(https://github.com/vllm-project/vllm/issues/31985):
    # There are longer-term solutions, like unwrapping the moe custom operator,
    # that aren't ready yet.
    # We could also treat the string as a "symbolic input" to the graph but
    # the PyTorch-side bits for that aren't ready yet either.
    #
    # If this value is None (like in some tests), then we end up baking the string
    # into the graph. Otherwise, the moe custom ops will pop a string from this list.
    all_moe_layers: list[str] | None = None
    moe_layer_index: int = 0

    additional_kwargs: dict[str, Any] = field(default_factory=dict)

    def __post_init__(self):
        assert self.cudagraph_runtime_mode.is_valid_runtime_mode(), (
            f"Invalid cudagraph runtime mode: {self.cudagraph_runtime_mode}"
        )
```
**EN:** `ForwardContext` is the per-pass carrier object. Besides attention metadata and slot mappings, it stores DP metadata, runtime CUDA-graph mode, microbatch slices, and a workaround for `torch.compile` cold starts: instead of baking MoE layer-name strings into graphs, operators consume them from `all_moe_layers` using `moe_layer_index`.
**CN:** `ForwardContext` 是单次前向过程的核心载体。除了注意力元数据和 slot 映射之外，它还保存 DP 元数据、运行时 CUDA graph 模式、microbatch 切片，以及一个专门为 `torch.compile` 冷启动准备的变通方案：MoE 层名字符串不直接硬编码进图，而是通过 `all_moe_layers` 和 `moe_layer_index` 按顺序读取。

### Context access and override helpers (lines 191-246)
```python
def get_forward_context() -> ForwardContext:
    """Get the current forward context."""
    assert _forward_context is not None, (
        "Forward context is not set. "
        "Please use `set_forward_context` to set the forward context."
    )
    return _forward_context


def is_forward_context_available() -> bool:
    return _forward_context is not None


def create_forward_context(
    attn_metadata: Any,
    vllm_config: VllmConfig,
    dp_metadata: DPMetadata | None = None,
    cudagraph_runtime_mode: CUDAGraphMode = CUDAGraphMode.NONE,
    batch_descriptor: BatchDescriptor | None = None,
    ubatch_slices: UBatchSlices | None = None,
    slot_mapping: dict[str, torch.Tensor] | list[dict[str, torch.Tensor]] | None = None,
    additional_kwargs: dict[str, Any] | None = None,
    skip_compiled: bool = False,
):
    if vllm_config.compilation_config.fast_moe_cold_start:
        all_moe_layers = vllm_config.compilation_config.static_all_moe_layers
    else:
        all_moe_layers = None

    return ForwardContext(
        no_compile_layers=vllm_config.compilation_config.static_forward_context,
        all_moe_layers=all_moe_layers,
        attn_metadata=attn_metadata,
        slot_mapping=slot_mapping or {},
        dp_metadata=dp_metadata,
        cudagraph_runtime_mode=cudagraph_runtime_mode,
        batch_descriptor=batch_descriptor,
        ubatch_slices=ubatch_slices,
        skip_compiled=skip_compiled,
        additional_kwargs=additional_kwargs or {},
    )


@contextmanager
def override_forward_context(forward_context: ForwardContext | None):
    """A context manager that overrides the current forward context.
    This is used to override the forward context for a specific
    forward pass.
    """
    global _forward_context
    prev_context = _forward_context
    _forward_context = forward_context
    try:
        yield
    finally:
        _forward_context = prev_context
```
**EN:** `get_forward_context()` enforces that consumers only run when a context is active, and `override_forward_context()` is the low-level context manager that swaps the module-global `_forward_context` in and out. This design keeps read access extremely cheap during hot inference paths.
**CN:** `get_forward_context()` 强制要求调用方只能在上下文已设置时访问它，`override_forward_context()` 则是底层上下文管理器，用来在模块全局变量 `_forward_context` 上做入栈/出栈切换。这种设计让推理热路径上的读取成本非常低。

### Main context manager orchestration (lines 249-356)
```python
@contextmanager
def set_forward_context(
    attn_metadata: Any,
    vllm_config: VllmConfig,
    num_tokens: int | None = None,
    num_tokens_across_dp: torch.Tensor | None = None,
    cudagraph_runtime_mode: CUDAGraphMode = CUDAGraphMode.NONE,
    batch_descriptor: BatchDescriptor | None = None,
    ubatch_slices: UBatchSlices | None = None,
    slot_mapping: dict[str, torch.Tensor] | list[dict[str, torch.Tensor]] | None = None,
    skip_compiled: bool = False,
):
    """A context manager that stores the current forward context,
    can be attention metadata, etc.
    Here we can inject common logic for every model forward pass.
    """
    global forward_start_time
    need_to_track_batchsize = track_batchsize and attn_metadata is not None
    if need_to_track_batchsize:
        forward_start_time = time.perf_counter()

    dp_metadata: DPMetadata | None = None
    if (
        vllm_config.parallel_config.data_parallel_size > 1
        and vllm_config.parallel_config.is_moe_model is not False
        and (attn_metadata is not None or num_tokens is not None)
    ):
        # If num_tokens_across_dp hasn't already been initialized, then
        # initialize it here. Both DP padding and Microbatching will be
        # disabled.
        if num_tokens_across_dp is None:
            assert ubatch_slices is None
            assert num_tokens is not None
            _, num_tokens_across_dp, _ = coordinate_batch_across_dp(
                num_tokens_unpadded=num_tokens,
                parallel_config=vllm_config.parallel_config,
                allow_microbatching=False,
            )
            assert num_tokens_across_dp is not None
        dp_metadata = DPMetadata.make(
            vllm_config.parallel_config, num_tokens or 0, num_tokens_across_dp
        )

    # Convenience: if cudagraph is used and num_tokens is given, we can just
    # create a batch descriptor here if not given (there's no harm since if it
    # doesn't match in the wrapper it'll fall through).
    if cudagraph_runtime_mode != CUDAGraphMode.NONE and num_tokens is not None:
        batch_descriptor = batch_descriptor or BatchDescriptor(num_tokens=num_tokens)

    additional_kwargs = current_platform.set_additional_forward_context(
        attn_metadata=attn_metadata,
        vllm_config=vllm_config,
        dp_metadata=dp_metadata,
        num_tokens=num_tokens,
        num_tokens_across_dp=num_tokens_across_dp,
        cudagraph_runtime_mode=cudagraph_runtime_mode,
        batch_descriptor=batch_descriptor,
        ubatch_slices=ubatch_slices,
    )

    forward_context = create_forward_context(
        attn_metadata,
        vllm_config,
        dp_metadata,
        cudagraph_runtime_mode,
        batch_descriptor,
        ubatch_slices,
        slot_mapping,
        additional_kwargs,
        skip_compiled,
    )

    try:
        with override_forward_context(forward_context):
            yield
    finally:
        global last_logging_time, batchsize_logging_interval
        if need_to_track_batchsize:
            batchsize = num_tokens
            # we use synchronous scheduling right now,
            # adding a sync point here should not affect
            # scheduling of the next batch
            synchronize = current_platform.synchronize
            if synchronize is not None:
                synchronize()
            now = time.perf_counter()
            # time measurement is in milliseconds
            batchsize_forward_time[batchsize].append((now - forward_start_time) * 1000)
            if now - last_logging_time > batchsize_logging_interval:
                last_logging_time = now
                forward_stats = []
                for bs, times in batchsize_forward_time.items():
                    if len(times) <= 1:
                        # can be cudagraph / profiling run
                        continue
                    medium = torch.quantile(torch.tensor(times), q=0.5).item()
                    medium = round(medium, 2)
                    forward_stats.append((bs, len(times), medium))
                forward_stats.sort(key=lambda x: x[1], reverse=True)
                if forward_stats:
                    logger.info(
                        (
                            "Batchsize forward time stats "
                            "(batchsize, count, median_time(ms)): %s"
                        ),
                        forward_stats,
                    )
```
**EN:** `set_forward_context()` is where the runtime policy lives. It optionally coordinates batch size across data-parallel ranks, auto-constructs a `BatchDescriptor` when CUDA graphs are active, asks `current_platform` for platform-specific additions, installs the resulting `ForwardContext`, and finally records median forward latency per batch size after synchronizing the device when needed.
**CN:** `set_forward_context()` 才是真正承载运行时策略的地方。它会在需要时协调数据并行各 rank 的 batch 大小、在启用 CUDA graph 时自动构造 `BatchDescriptor`、向 `current_platform` 请求平台特定补充字段、安装最终的 `ForwardContext`，并在必要时同步设备后记录每个 batch size 的前向中位耗时。

## Key Concepts / 关键概念
- EN: A module-global context is used because many kernels and helper layers need cheap ambient access during a single forward pass.
- CN: 之所以采用模块级全局上下文，是因为许多内核和辅助层都需要在一次前向过程中以极低成本获取环境信息。
- EN: The file mixes execution metadata with telemetry; timing is optional and deliberately kept off unless configured.
- CN: 该文件同时管理执行元数据和遥测信息；耗时统计是可选的，只有显式配置后才会启用。
- EN: CUDA-graph selection is shape-aware but intentionally compact, using `BatchDescriptor` instead of a full request object.
- CN: CUDA graph 的选择依赖形状信息，但刻意保持紧凑，只使用 `BatchDescriptor` 而不是完整请求对象。

## Dependencies / 依赖关系
- EN: Depends on PyTorch tensors plus vLLM config types like `VllmConfig`, `ParallelConfig`, and `CUDAGraphMode`.
- CN: 依赖 PyTorch 张量以及 `VllmConfig`、`ParallelConfig`、`CUDAGraphMode` 等 vLLM 配置类型。
- EN: Uses `current_platform` hooks and DP helpers such as `coordinate_batch_across_dp`.
- CN: 依赖 `current_platform` 提供的平台钩子以及 `coordinate_batch_across_dp` 等数据并行辅助函数。
- EN: Attention metadata types come from `vllm.v1.attention.backend`, and microbatch slicing comes from `UBatchSlices`.
- CN: 注意力元数据类型来自 `vllm.v1.attention.backend`，microbatch 切片则来自 `UBatchSlices`。

# env_override.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `vllm/env_override.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Applies import-time environment defaults and version-scoped PyTorch/Inductor monkeypatches needed by vLLM. / 在导入阶段设置环境默认值，并应用 vLLM 所需的、按 PyTorch 版本分流的兼容性补丁。

## Line-by-Line Analysis / 逐行分析
### Pre-import CUDA compatibility path setup (lines 8-85)
```python
def _get_torch_cuda_version():
    """Peripheral function to _maybe_set_cuda_compatibility_path().
    PyTorch version must not be determined by importing directly
    because it will trigger the CUDA initialization, losing the
    chance to set the LD_LIBRARY_PATH beforehand.
    """
    try:
        spec = importlib.util.find_spec("torch")
        if not spec:
            return None
        if spec.origin:
            torch_root = os.path.dirname(spec.origin)
        elif spec.submodule_search_locations:
            torch_root = spec.submodule_search_locations[0]
        else:
            return None
        version_path = os.path.join(torch_root, "version.py")
        if not os.path.exists(version_path):
            return None
        # Load the version module without importing torch
        ver_spec = importlib.util.spec_from_file_location("torch.version", version_path)
        if not ver_spec or not ver_spec.loader:
            return None
        module = importlib.util.module_from_spec(ver_spec)
        # Avoid registering in sys.modules to not confuse future imports
        ver_spec.loader.exec_module(module)
        return getattr(module, "cuda", None)
    except Exception:
        return None


def _maybe_set_cuda_compatibility_path():
    """Set LD_LIBRARY_PATH for CUDA forward compatibility if enabled.

    Must run before 'import torch' since torch loads CUDA shared libraries
    at import time and the dynamic linker only consults LD_LIBRARY_PATH when
    a library is first loaded.

    CUDA forward compatibility is only supported on select professional and
    datacenter NVIDIA GPUs. Consumer GPUs (GeForce, RTX) do not support it
    and will get Error 803 if compat libs are loaded.
    """
    enable = os.environ.get("VLLM_ENABLE_CUDA_COMPATIBILITY", "0").strip().lower() in (
        "1",
        "true",
    )
    if not enable:
        return

    cuda_compat_path = os.environ.get("VLLM_CUDA_COMPATIBILITY_PATH", "")
    if not cuda_compat_path or not os.path.isdir(cuda_compat_path):
        conda_prefix = os.environ.get("CONDA_PREFIX", "")
        conda_compat = os.path.join(conda_prefix, "cuda-compat")
        if conda_prefix and os.path.isdir(conda_compat):
            cuda_compat_path = conda_compat
    if not cuda_compat_path or not os.path.isdir(cuda_compat_path):
        torch_cuda_version = _get_torch_cuda_version()
        if torch_cuda_version:
            default_path = f"/usr/local/cuda-{torch_cuda_version}/compat"
            if os.path.isdir(default_path):
                cuda_compat_path = default_path
    if not cuda_compat_path or not os.path.isdir(cuda_compat_path):
        return

    norm_path = os.path.normpath(cuda_compat_path)
    existing = os.environ.get("LD_LIBRARY_PATH", "")
    ld_paths = existing.split(os.pathsep) if existing else []

    if ld_paths and ld_paths[0] and os.path.normpath(ld_paths[0]) == norm_path:
        return  # Already at the front

    new_paths = [norm_path] + [
        p for p in ld_paths if not p or os.path.normpath(p) != norm_path
    ]
    os.environ["LD_LIBRARY_PATH"] = os.pathsep.join(new_paths)


_maybe_set_cuda_compatibility_path()
```
**EN:** The first critical trick is that vLLM discovers Torch's CUDA version without importing `torch`, then prepends a compatibility library directory to `LD_LIBRARY_PATH` before any CUDA shared objects are loaded. That ordering matters because the dynamic linker only consults `LD_LIBRARY_PATH` on first load.
**CN:** 这里最关键的技巧是：vLLM 在不导入 `torch` 的前提下读取 Torch 的 CUDA 版本，并在任何 CUDA 共享库加载前，把兼容库目录插到 `LD_LIBRARY_PATH` 前面。这个顺序非常重要，因为动态链接器只会在首次加载库时读取 `LD_LIBRARY_PATH`。

### Import-time environment defaults (lines 94-120)
```python
# set some common config/environment variables that should be set
# for all processes created by vllm and all processes
# that interact with vllm workers.
# they are executed whenever `import vllm` is called.

# see https://github.com/vllm-project/vllm/pull/15951
# it avoids unintentional cuda initialization from torch.cuda.is_available()
os.environ["PYTORCH_NVML_BASED_CUDA_CHECK"] = "1"

# see https://github.com/vllm-project/vllm/issues/10480 and
# https://github.com/vllm-project/vllm/issues/10619.
os.environ["TORCHINDUCTOR_COMPILE_THREADS"] = "1"

# Enable Triton autotuning result caching to disk by default.
# Without this, Triton re-runs autotuning on every process restart,
# adding significant latency to the first inference request.
# This writes autotuning results to TRITON_CACHE_DIR.
# It can still be overridden by setting TRITON_CACHE_AUTOTUNING=0
# in the environment.
os.environ.setdefault("TRITON_CACHE_AUTOTUNING", "1")

# When unset, TileLang routes JIT temp dirs through a world-shared
# /tmp/tvm-debug-mode-tempdirs/ whose ownership is pinned to whichever
# user compiled first, breaking every other user on a shared host.
# Opt into per-process tempdirs unless the user explicitly chose the
# debug layout (see https://github.com/vllm-project/vllm/issues/41410).
os.environ.setdefault("TILELANG_CLEANUP_TEMP_FILES", "1")
```
**EN:** These assignments are global process defaults executed as part of `import vllm`: avoid accidental CUDA initialization (`PYTORCH_NVML_BASED_CUDA_CHECK=1`), cap Inductor compile threads, enable Triton autotune caching, and force TileLang away from a world-shared temp directory that breaks multi-user hosts.
**CN:** 这些赋值是在 `import vllm` 期间执行的全局进程默认值：避免误触发 CUDA 初始化（`PYTORCH_NVML_BASED_CUDA_CHECK=1`）、限制 Inductor 编译线程数、启用 Triton autotune 磁盘缓存，并让 TileLang 避开多用户主机上会出问题的共享临时目录。

### Torch 2.9 memory planning patch (lines 130-186)
```python
def memory_plan_reuse_patched(self):
    import torch._inductor.ir as ir
    from torch._inductor.codegen.wrapper import (
        EnterSubgraphLine,
        ExitSubgraphLine,
        MemoryPlanningLine,
        MemoryPlanningState,
        SubgraphPythonWrapperCodegen,
    )
    from torch._inductor.virtualized import V

    def get_output_names(graph_outputs) -> list[str]:
        import itertools

        names = []
        shape_counter = itertools.count(0)
        none_counter = itertools.count(0)
        for node in graph_outputs:
            if isinstance(node, ir.NoneAsConstantBuffer):
                names.append(f"{V.graph.name}_none{next(none_counter)}")
            elif isinstance(node, ir.ShapeAsConstantBuffer):
                names.append(f"{V.graph.name}_shape{next(shape_counter)}")
            else:
                names.append(node.get_name())
        return names

    if (
        isinstance(V.graph.wrapper_code, SubgraphPythonWrapperCodegen)
        and V.graph.wrapper_code.partition_signatures is not None
    ):
        out_names = get_output_names(
            V.graph.wrapper_code.partition_signatures.output_nodes
        )
    else:
        out_names = V.graph.get_output_names()

    while (
        self.lines
        and isinstance(self.lines[-1], MemoryPlanningLine)
        and self.lines[-1].node.name not in out_names  # type: ignore[attr-defined]
    ):
        # these lines will be pointless
        self.lines.pop()

    # codegen allocations in two passes
    planning_states = [MemoryPlanningState()]
    past_planning_states = []
    for i in range(len(self.lines)):
        line = self.lines[i]
        if isinstance(line, MemoryPlanningLine):
            self.lines[i] = line.plan(planning_states[-1])
        elif isinstance(line, EnterSubgraphLine):
            planning_states.append(MemoryPlanningState())
        elif isinstance(line, ExitSubgraphLine):
            past_planning_states.append(planning_states.pop())
    past_planning_states.append(planning_states.pop())
    assert len(planning_states) == 0
```
**EN:** `memory_plan_reuse_patched()` backports a fix for piecewise graph compilation by computing output names carefully, including synthetic names for `NoneAsConstantBuffer` and `ShapeAsConstantBuffer`. It then drops useless trailing `MemoryPlanningLine` entries and recomputes memory planning states in two passes.
**CN:** `memory_plan_reuse_patched()` 为分段图编译回补了一个修复：它会精确计算输出名，包括 `NoneAsConstantBuffer` 与 `ShapeAsConstantBuffer` 这类合成输出名。之后它会删除无意义的尾部 `MemoryPlanningLine`，并分两遍重新规划内存状态。

### Torch 2.9 graph partition signature patch (lines 198-336)
```python
def get_graph_partition_signature_patched(
    self, partitions, skip_cudagraphs: list[bool]
):
    """
    Gets signature for each graph partition, including input nodes, output nodes, and
    whether deallocating an input within graph partition.
    """
    from torch._inductor import dependencies
    from torch._inductor.ir import GraphPartitionSignature, MutationOutput, NoneLayout
    from torch._inductor.virtualized import V
    from torch.utils._ordered_set import OrderedSet

    signatures = []

    unmet_output_names = OrderedSet(V.graph.get_output_names())
    name_to_node = self.get_name_to_nodes()

    def is_none_layout(buf_name: str) -> bool:
        """
        Checks if buf_name is NoneLayout. Buffers with NoneLayout is not allocated
        so graph partition should not take it as inputs or outputs.
        """
        buf = self.name_to_buf.get(buf_name, None)

        if buf is None:
            return False

        if isinstance(buf.node.layout, NoneLayout):
            if isinstance(buf.node, MutationOutput) and (
                real_name := self.mutation_real_name.get(buf_name, None)
            ):
                return is_none_layout(real_name)

            return True

        return False

    for partition, skip_cudagraph in zip(
        reversed(partitions), reversed(skip_cudagraphs)
    ):
        output_names: OrderedSet[str] = OrderedSet()

        for node in partition:
            output_names.update(node.outputs_by_name.keys())

        returned_output_names = output_names.intersection(unmet_output_names)

        # all reads/writes are partition inputs except those generated
        # within the partition and tensor constants
        read_writes = dependencies.ReadWrites.merge_list(
            [node.read_writes for node in partition]
        )

        # WeakDep is fake dependency on unused buffer. It should not appear
        # in partition_input_names for inputs that are actually read or written.
        partition_input_names = (
            OrderedSet(
                [
                    x.name
                    for x in read_writes.reads | read_writes.writes
                    if not is_none_layout(x.name)
                ]
            )
            - output_names
        )

        partition_input_names = OrderedSet(
            self.mutation_real_name.get(name, name) for name in partition_input_names
        )

        buffer_names_to_free: OrderedSet[str] = OrderedSet()
        for node in partition:
            buffer_names_to_free.update(node.last_usage)

        # buffer_names_to_free may contain buffers allocated in previous
        # graph partitions. These buffers should also be a partition
        # input.
        extra_input_names = [
            name
            for name in (buffer_names_to_free - output_names)
            if name in name_to_node
        ]
        partition_input_names.update(extra_input_names)

        input_nodes = {
            name: name_to_node[name]
            for name in partition_input_names
            if name in name_to_node
        }
        input_deallocation = {
            name: name in buffer_names_to_free
            for name in partition_input_names
            if name in name_to_node
        }

        # if an input tensor is not freed in the partition function, it should
        # also be returned as an output. This brings benefits to cudagraph
        # since the returned output tensor is a cudagraph managed tensor with
        # a static tensor address.
        extra_output_names = [
            name
            for name in partition_input_names
            if name in name_to_node and name not in buffer_names_to_free
        ]

        returned_output_names.update(extra_output_names)

        returned_output_names = OrderedSet(
            self.mutation_real_name.get(name, name) for name in returned_output_names
        )

        output_nodes = [
            name_to_node[name]
            for name in returned_output_names
            if not is_none_layout(name)
        ]

        constant_names = [
            name for name in partition_input_names if name in V.graph.constants
        ]

        symbol_inputs = self.get_graph_partition_symbol_inputs(partition, input_nodes)

        partition_signature = GraphPartitionSignature(
            symbol_inputs,
            input_nodes,
            output_nodes,
            input_deallocation,
            skip_cudagraph,
            constant_names,
        )

        signatures.append(partition_signature)

        unmet_output_names = partition_input_names.union(
            unmet_output_names - returned_output_names
        )

    return signatures[::-1]
```
**EN:** This patched signature builder is about preserving correct inputs/outputs around partition boundaries. It removes `NoneLayout` buffers, tracks buffers freed inside a partition, promotes surviving inputs into outputs so CUDA graphs keep stable addresses, and preserves constant names and symbolic inputs for Inductor.
**CN:** 这个补丁版签名构建器的核心是保证分区边界上的输入/输出正确性。它会排除 `NoneLayout` 缓冲区、跟踪分区内被释放的 buffer、把仍需保留的输入提升为输出，以便 CUDA graph 维持稳定地址，同时保留常量名与符号输入给 Inductor。

### Scheduler and raw stream workarounds (lines 360-499)
```python
def should_partition_patched(self, node, should_log: bool = False) -> bool:
    # This is a patched version of
    # torch._inductor.scheduler.Scheduler.should_partition that modifies
    # the following piece of code so that we always return True:
    # https://github.com/pytorch/pytorch/blob/ecb53078faf86ca1b33277df33b82985675bb011/torch/_inductor/scheduler.py#L4712-L4724
    """Return True if we should partition the inductor graph on this node"""

    import torch._inductor.ir as ir
    from torch._inductor.scheduler import (
        BaseSchedulerNode,
        FusedSchedulerNode,
    )
    from torch._inductor.utils import (
        _unstable_customized_partition_wrapper,
        is_cudagraph_unsafe_op,
        maybe_log_cudagraph_partition,
    )

    # Allow users to manually specify if a node should be partitioned
    # Can only do this for FallbackKernels
    ir_node = node.node
    if isinstance(ir_node, torch._inductor.ir.FallbackKernel) and (
        op := ir_node.op_overload
    ):
        op_overload_packet_name = op.name()
        op_overload_name = (
            f"{op_overload_packet_name}.{op._overloadname}"
            if isinstance(op, torch._ops.OpOverload)
            else op_overload_packet_name
        )
        if (
            op_overload_packet_name
            in torch._inductor.config.custom_should_partition_ops
            or op_overload_name in torch._inductor.config.custom_should_partition_ops
        ):
            assert isinstance(op, torch._ops.OpOverload)
            return True

    # When not using cudagraphs, keep all kernels in the `call` function
    # instead of graph partition functions, since graph partition only brings
    # benefit to cudagraph
    if (
        not torch._inductor.config.triton.cudagraphs
        and _unstable_customized_partition_wrapper.wrapper is None
    ):
        return True

    # avoid duplicating logs when should_partition is called multiple times
    # on the same node
    def noop_log(msg: str, node: BaseSchedulerNode | None) -> None:
        return

    log_partition_reason = maybe_log_cudagraph_partition if should_log else noop_log

    if isinstance(node, FusedSchedulerNode):
        return any(self.should_partition(snode) for snode in node.snodes)

    assert node.node is not None

    if not node.is_gpu():
        log_partition_reason("non gpu ops", node=node)

        return True

    if isinstance(node.node, ir.DeviceCopy):
        log_partition_reason("DeviceCopy ops", node=node)
        return True

    if isinstance(node.node, ir.Conditional):
        log_partition_reason("Conditional ops", node=node)
        return True

    if getattr(node.node, "unbacked_bindings", None):
        log_partition_reason("unbacked binding ops", node=node)
        return True

    if is_cudagraph_unsafe_op(node.node):
        log_partition_reason("CUDAGraph-unsafe custom ops", node=node)
        return True

    return False


def _update_scheduler_patched(self) -> None:
    # Copied from torch._inductor.graph.GrahLowering._update_scheduler. Patches
    # this method so that we can patch Scheduler.should_partition with the
    # function above
    """
    (Re)initializes the scheduler member.  When initializing the scheduler, no CUBIN
    files should be generated (to avoid biasing any benchmarks and pessimizing
    fusion decisions).
    """
    import torch._inductor.config as config
    from torch._inductor.scheduler import Scheduler

    Scheduler.should_partition = should_partition_patched
    Scheduler.get_graph_partition_signature = get_graph_partition_signature_patched

    with config.patch("triton.store_cubin", False):
        self.scheduler = Scheduler(self.operations)


# ===================================================
# torch 2.9 Inductor get_raw_stream workaround
# ===================================================
# Workaround for TorchInductor autotune using get_raw_stream() without defining it.
# This occurs when compile_sizes > 1 in compilation_config.
# For more context, see https://github.com/vllm-project/vllm/issues/30905.
def _patch_get_raw_stream_if_needed():
    """Workaround for TorchInductor autotune get_raw_stream() bug."""
    from vllm.utils.torch_utils import is_torch_equal

    # Only apply the patch for torch 2.9.0 or 2.9.1
    if is_torch_equal("2.9.0") or is_torch_equal("2.9.1"):
        import builtins

        # Check if CUDA functionality is available without initializing CUDA
        # _cuda_getCurrentRawStream only exists in CUDA builds of PyTorch
        if hasattr(torch._C, "_cuda_getCurrentRawStream"):
            from torch._C import _cuda_getCurrentRawStream as _get_raw_stream

            builtins.get_raw_stream = _get_raw_stream  # type: ignore[attr-defined]


_patch_get_raw_stream_if_needed()

if is_torch_equal("2.9.0"):
    from torch._inductor.codegen.wrapper import PythonWrapperCodegen
    from torch._inductor.graph import GraphLowering
    from torch.utils._config_module import _Config, _ConfigEntry

    # `custom_should_partition_ops` is a new config after 2.9.0. So this would
    # not overwrite any user configs.
    torch._inductor.config._config["custom_should_partition_ops"] = _ConfigEntry(
        _Config(default=[])
    )

    PythonWrapperCodegen.memory_plan_reuse = memory_plan_reuse_patched
    GraphLowering._update_scheduler = _update_scheduler_patched

```
**EN:** `should_partition_patched()` avoids an Inductor crash path where certain custom/fallback ops lack `origin_node`, while `_update_scheduler_patched()` injects that patch plus the partition-signature fix into scheduler creation. `_patch_get_raw_stream_if_needed()` then plugs a separate autotune bug by exporting `_cuda_getCurrentRawStream` as `builtins.get_raw_stream` on affected 2.9 releases.
**CN:** `should_partition_patched()` 避开了 Inductor 的一个崩溃路径：某些自定义/回退算子缺少 `origin_node` 时原始逻辑会出错；`_update_scheduler_patched()` 则在构造调度器时一并注入这个补丁与分区签名补丁。随后 `_patch_get_raw_stream_if_needed()` 又修复了另一个 autotune 问题：在受影响的 2.9 版本上，把 `_cuda_getCurrentRawStream` 暴露为 `builtins.get_raw_stream`。

### Graph lowering and AOT runtime env patches (lines 522-594)
```python
def _apply_constrain_to_fx_strides_patch():
    """Patch lowering.constrain_to_fx_strides globally. Safe to call
    multiple times; only the first call does anything.
    Only applies for torch >= 2.11 and < 2.12."""
    global _constrain_to_fx_strides_patched
    if _constrain_to_fx_strides_patched:
        return
    _constrain_to_fx_strides_patched = True

    if not is_torch_equal_or_newer("2.11.0.dev") or is_torch_equal_or_newer(
        "2.12.0.dev"
    ):
        return

    import torch._inductor.ir as _ir
    import torch._inductor.lowering as _lowering
    from torch._inductor.virtualized import V as _V

    def _patched(fx_node, *args, **kwargs):
        def apply_constraint(arg, fx_arg):
            if isinstance(arg, _ir.IRNode):
                meta_val = fx_arg.meta.get("val")
                if isinstance(meta_val, torch.Tensor):
                    stride_order = _ir.get_stride_order(
                        meta_val.stride(), _V.graph.sizevars.shape_env
                    )
                    return _ir.ExternKernel.require_stride_order(arg, stride_order)
                return arg
            if isinstance(arg, dict):
                return {key: apply_constraint(arg[key], fx_arg[key]) for key in arg}
            return arg

        args = tuple(
            apply_constraint(arg, fx_arg) for arg, fx_arg in zip(args, fx_node.args)
        )
        kwargs = {k: apply_constraint(v, fx_node.kwargs[k]) for k, v in kwargs.items()}
        return args, kwargs

    _lowering.constrain_to_fx_strides = _patched


if is_torch_equal_or_newer("2.10.0") and not is_torch_equal_or_newer("2.12.0.dev"):
    import builtins as _builtins
    import pickle

    from torch._dynamo.convert_frame import GraphCaptureOutput

    _original_get_runtime_env = GraphCaptureOutput.get_runtime_env

    def _safe_builtins_dict(builtins_dict: dict) -> dict:
        """Filter a builtins dict to only picklable entries for serialization."""
        result = {}
        for k, v in builtins_dict.items():
            try:
                pickle.dumps(v)
                result[k] = v
            except Exception:
                pass
        return result

    def _patched_get_runtime_env(self):  # type: ignore[no-untyped-def]
        runtime_env = _original_get_runtime_env(self)
        for ref in runtime_env.external_refs:
            if ref not in runtime_env.used_globals:
                if ref.startswith("__builtins_dict__") and ref in self.f_globals:
                    runtime_env.used_globals[ref] = _safe_builtins_dict(
                        self.f_globals[ref]
                    )
                elif hasattr(_builtins, ref):
                    runtime_env.used_globals[ref] = getattr(_builtins, ref)
        return runtime_env

    GraphCaptureOutput.get_runtime_env = _patched_get_runtime_env
```
**EN:** `_apply_constrain_to_fx_strides_patch()` skips stride inference for non-tensor FX metadata, which avoids crashes on `FakeScriptObject`. The `GraphCaptureOutput.get_runtime_env` patch fixes missing builtins in AOT serialization by copying either safe picklable builtin dictionaries or named builtin symbols into `used_globals`.
**CN:** `_apply_constrain_to_fx_strides_patch()` 会在 FX 元数据不是张量时跳过 stride 推断，从而避免 `FakeScriptObject` 触发崩溃。`GraphCaptureOutput.get_runtime_env` 的补丁则修复 AOT 序列化中内建对象缺失的问题：它会把安全、可 pickle 的 builtin 字典或具名 builtin 符号补回 `used_globals`。

### FxGraphCache pickling workaround (lines 606-643)
```python
def _apply_fxgraphcache_pickle_patch(pickler_cls, bypass_cls):
    """Wrap pickler_cls.dumps to convert ValueError into bypass_cls.

    Idempotent: sets `_vllm_fxgraph_dumps_patched` on the class after the
    first apply to prevent re-application. The wrapper function is also
    marked with `_vllm_patched` as an additional safeguard.
    """
    if getattr(pickler_cls, "_vllm_fxgraph_dumps_patched", False):
        return

    original_dumps = pickler_cls.dumps
    if hasattr(original_dumps, "_vllm_patched"):
        return

    def patched_dumps(self, obj):
        try:
            return original_dumps(self, obj)
        except ValueError as e:
            raise bypass_cls("Failed to pickle cache key") from e

    patched_dumps._vllm_patched = True  # type: ignore[attr-defined]
    pickler_cls.dumps = patched_dumps
    pickler_cls._vllm_fxgraph_dumps_patched = True  # type: ignore[attr-defined]


def _patch_fxgraphcache_pickle_if_needed():
    """Apply FxGraphCachePickler.dumps ValueError backport when on torch 2.10.x."""
    from vllm.utils.torch_utils import is_torch_equal_or_newer

    if not is_torch_equal_or_newer("2.10.0") or is_torch_equal_or_newer("2.11.0"):
        return

    from torch._inductor.codecache import BypassFxGraphCache, FxGraphCachePickler

    _apply_fxgraphcache_pickle_patch(FxGraphCachePickler, BypassFxGraphCache)


_patch_fxgraphcache_pickle_if_needed()
```
**EN:** PyTorch 2.10 can raise `ValueError` while serializing unusual tensor layouts into the compile cache key. vLLM wraps `FxGraphCachePickler.dumps()` so those failures become `BypassFxGraphCache`, meaning compilation still proceeds but that graph skips the broken cache path instead of crashing.
**CN:** PyTorch 2.10 在把某些非常规张量布局序列化到编译缓存键时，可能抛出 `ValueError`。vLLM 对 `FxGraphCachePickler.dumps()` 做了一层包装，把这类失败转换为 `BypassFxGraphCache`：这样编译仍可继续，只是该图绕过有问题的缓存路径，而不是直接崩溃。

### Deferred C++ codegen patching (lines 660-760)
```python
def _apply_cpp_indirect_assert_patch():
    """Replace CppVecKernel.indirect_assert with a fixed copy that uses
    `VecMask<...>::from(scalar)` for scalar masks.

    Idempotent: marks the class with `_vllm_indirect_assert_patched` after
    the first apply.
    """
    from torch._inductor.codegen.cpp import CppVecKernel

    if getattr(CppVecKernel, "_vllm_indirect_assert_patched", False):
        return

    from torch._inductor.codegen.cpp import CppCSEVariable, cexpr_index

    def patched_indirect_assert(self, var, lower, upper, mask=None):
        assert isinstance(var, CppCSEVariable)
        assert var.dtype is not None
        if not var.is_vec:
            if isinstance(mask, CppCSEVariable) and mask.is_vec:
                mask = f"({mask}).all_masked()"
            return super(CppVecKernel, self).indirect_assert(var, lower, upper, mask)
        lower_scalar = lower
        upper_scalar = upper
        if lower:
            lower = f"{self._get_vec_type(var.dtype)}({lower})"
        if upper:
            upper = f"{self._get_vec_type(var.dtype)}({upper})"
        if lower and upper:
            cond = f"({lower} <= {var}) & ({var} < {upper})"
            cond_print = f"{lower_scalar} <= {var} < {upper_scalar}"
        elif lower:
            cond = f"{lower} <= {var}"
            cond_print = f"{lower_scalar} <= {var}"
        else:
            assert upper
            cond = f"{var} < {upper}"
            cond_print = f"{var} < {upper_scalar}"
        cond = f"{self._get_mask_type(var.dtype)}({cond})"
        if mask:
            if not mask.is_vec:
                # Backport of pytorch/pytorch#178148 -- use ::from for
                # scalar masks so g++ picks the correct overload.
                mask = f"{self._get_mask_type(var.dtype)}::from({mask})"
            cond = f"({cond}) | ~({mask})"
        if self.tail_size:
            cond = (
                f"{self._get_mask_type(var.dtype)}::set("
                f"{self._get_mask_type(var.dtype)}::from(1)"
                f", ({cond}), {cexpr_index(self.tail_size)})"
            )
        cond = f"({cond}).all_masked()"
        return f'{self.assert_function}({cond}, "index out of bounds: {cond_print}")'

    CppVecKernel.indirect_assert = patched_indirect_assert
    CppVecKernel._vllm_indirect_assert_patched = True  # type: ignore[attr-defined]


def _patch_cpp_indirect_assert_if_needed():
    """Apply cpp codegen indirect_assert backport when on torch 2.11.x.

    Defers application until torch._inductor.codegen.cpp is naturally
    imported by Inductor. Importing it eagerly during vllm.__init__ pulls
    in torch._inductor.scheduler, whose top-level
    `import torch._inductor.async_compile` can fail with
    `ModuleNotFoundError: import of torch._inductor.async_compile halted;
    None in sys.modules` depending on the import order on the runner
    (observed in vLLM CPU CI).
    """
    if not is_torch_equal_or_newer("2.11.0") or is_torch_equal_or_newer("2.12.0.dev"):
        return

    import sys

    target_name = "torch._inductor.codegen.cpp"
    if target_name in sys.modules:
        _apply_cpp_indirect_assert_patch()
        return

    import importlib.abc

    class _CppCodegenPatchFinder(importlib.abc.MetaPathFinder):
        def find_spec(self, fullname, path, target=None):
            if fullname != target_name:
                return None
            sys.meta_path.remove(self)
            spec = importlib.util.find_spec(fullname)
            if spec is None or spec.loader is None:
                return None
            original_exec = spec.loader.exec_module

            def _exec_then_patch(module):
                original_exec(module)
                _apply_cpp_indirect_assert_patch()

            spec.loader.exec_module = _exec_then_patch  # type: ignore[method-assign]
            return spec

    sys.meta_path.insert(0, _CppCodegenPatchFinder())


_patch_cpp_indirect_assert_if_needed()
```
**EN:** The indirect-assert patch changes scalar mask construction from an invalid constructor call to `VecMask::from(...)`, fixing generated C++ for some vectorized kernels. Importantly, vLLM may delay this patch with a custom `MetaPathFinder` so it is applied exactly when `torch._inductor.codegen.cpp` is naturally imported, avoiding fragile eager-import side effects during startup.
**CN:** 这个 indirect-assert 补丁把标量 mask 的构造方式从无效构造函数改为 `VecMask::from(...)`，从而修复某些向量化内核生成出的 C++ 代码。更重要的是，vLLM 可能通过自定义 `MetaPathFinder` 延迟应用该补丁，只在 `torch._inductor.codegen.cpp` 被自然导入时才执行，避免启动阶段提前导入带来的脆弱副作用。

## Key Concepts / 关键概念
- EN: This module is intentionally side-effectful: importing it changes process environment and Torch internals.
- CN: 该模块是有意设计成“带副作用”的：一旦导入，就会修改进程环境和 Torch 内部行为。
- EN: Most patches are narrowly version-gated, which keeps vLLM compatible with multiple Torch releases without permanently forking Inductor code.
- CN: 大多数补丁都按版本精确分流，因此 vLLM 可以兼容多个 Torch 版本，而无需长期维护一整套分叉的 Inductor 代码。
- EN: Several fixes prefer graceful degradation—skip cache, defer import, or patch builtins—instead of failing hard at import time.
- CN: 多个修复都偏向“优雅降级”——跳过缓存、延迟导入、补齐 builtins——而不是在导入时直接硬失败。

## Dependencies / 依赖关系
- EN: Deeply depends on `torch`, especially `torch._inductor`, `torch._dynamo`, and generated code paths.
- CN: 深度依赖 `torch`，尤其是 `torch._inductor`、`torch._dynamo` 与代码生成相关路径。
- EN: Uses `vllm.logger.init_logger` and version predicates from `vllm.utils.torch_utils` to apply patches only when needed.
- CN: 使用 `vllm.logger.init_logger` 和 `vllm.utils.torch_utils` 中的版本判断函数，仅在需要时应用补丁。
- EN: Relies on environment variables defined in `vllm.envs`, especially CUDA compatibility knobs.
- CN: 依赖 `vllm.envs` 中定义的环境变量，尤其是 CUDA 兼容性相关开关。

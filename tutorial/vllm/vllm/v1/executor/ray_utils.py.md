# ray_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/executor/ray_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `detach_zero_copy_from_model_runner_output`, `FutureWrapper`, `ray_is_available` for the V1 `executor` subsystem. / 为 V1 的 `executor` 子系统实现 `detach_zero_copy_from_model_runner_output`, `FutureWrapper`, `ray_is_available`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import os
import time
from collections import defaultdict
from concurrent.futures import Future
from typing import TYPE_CHECKING, Union

import numpy as np

import vllm.platforms
from vllm.config import ParallelConfig
from vllm.distributed import get_pp_group
from vllm.distributed.kv_transfer.kv_connector.utils import KVOutputAggregator
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.sequence import IntermediateTensors
from vllm.utils.network_utils import get_ip
from vllm.v1.outputs import AsyncModelRunnerOutput
from vllm.v1.serial_utils import run_method
from vllm.v1.worker.worker_base import WorkerWrapperBase

if TYPE_CHECKING:
    from vllm.v1.core.sched.output import GrammarOutput, SchedulerOutput
    from vllm.v1.outputs import ModelRunnerOutput

logger = init_logger(__name__)
PG_WAIT_TIMEOUT = 1800

# Env vars that are worker-specific and must NOT be copied from the
# driver to Ray workers — they are set per-worker after GPU discovery.
WORKER_SPECIFIC_ENV_VARS: set[str] = {
    "VLLM_HOST_IP",
    "VLLM_HOST_PORT",
    "VLLM_NIXL_SIDE_CHANNEL_HOST",
    "LOCAL_RANK",
    "CUDA_VISIBLE_DEVICES",
    "HIP_VISIBLE_DEVICES",
    "ROCR_VISIBLE_DEVICES",
}

try:
    import ray
    from ray.util import placement_group_table
    from ray.util.placement_group import PlacementGroup

    try:
        from ray._private.state import available_resources_per_node
    except ImportError:
        # Ray 2.9.x doesn't expose `available_resources_per_node`
        from ray._private.state import state as _state

# ... omitted for brevity ...
                output = self.worker.model_runner.sample_tokens(grammar_output)
                # Ensure outputs crossing Ray compiled DAG are serializable.
                # AsyncModelRunnerOutput holds CUDA events and cannot be
                # pickled.
                if isinstance(output, AsyncModelRunnerOutput):
                    output = output.get_output()
            return output

        def override_env_vars(self, vars: dict[str, str]):
            os.environ.update(vars)

        def _is_intermediate_tensors(self, output) -> bool:
            return isinstance(output, IntermediateTensors)

        def _is_last_rank(self) -> bool:
            return get_pp_group().is_last_rank

    ray_import_err = None

except ImportError as e:
    ray = None  # type: ignore
    # only capture string to avoid variable references in the traceback that can
    # prevent garbage collection in some cases
    ray_import_err = str(e)
    RayWorkerWrapper = None  # type: ignore
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `PG_WAIT_TIMEOUT`, `WORKER_SPECIFIC_ENV_VARS`. This excerpt omits repetitive declarations.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `PG_WAIT_TIMEOUT`, `WORKER_SPECIFIC_ENV_VARS`。 该片段省略了重复性声明。

### `detach_zero_copy_from_model_runner_output` function / `detach_zero_copy_from_model_runner_output` 函数
```python
def detach_zero_copy_from_model_runner_output(output: "ModelRunnerOutput") -> None:
    """Detach Ray SHM-channel zero-copy buffers from a ModelRunnerOutput in-place.

    Ray compiled DAG SHM channels may return zero-copy objects (e.g. `np.ndarray`)
    backed by Ray's shared-memory object store. Ray's channel docs explicitly
    warn that subsequent reads may block if such an object is still in scope.

    vLLM can return numpy-backed logprobs in `ModelRunnerOutput.logprobs`. If
    those arrays are backed by Ray SHM (commonly read-only), retaining them in
    scope across scheduler iterations can stall the channel and eventually hit
    `RAY_CGRAPH_get_timeout`.

    Copy read-only numpy arrays so the returned output no longer retains
    references to Ray's shared-memory buffers.

    We intentionally do not touch `prompt_logprobs_dict`: those entries are
    `LogprobsTensors` backed by PyTorch-owned CPU tensors (`to_cpu_nonblocking`
    or `empty_cpu`), not NumPy views decoded from Ray channels.
    """
    if output.logprobs is None:
        return

    token_ids, logprobs, ranks, cu_num_generated_tokens = output.logprobs

    def _copy_if_readonly(arr):
        if isinstance(arr, np.ndarray) and not arr.flags.writeable:
            return arr.copy()
        return arr

    # `cu_num_generated_tokens` is already a plain Python list (or None), so it
    # never aliases Ray SHM buffers and can be reused as-is.
    token_ids_c = _copy_if_readonly(token_ids)
    logprobs_c = _copy_if_readonly(logprobs)
    ranks_c = _copy_if_readonly(ranks)
    if token_ids_c is token_ids and logprobs_c is logprobs and ranks_c is ranks:
        return

    output.logprobs = type(output.logprobs)(
        token_ids_c, logprobs_c, ranks_c, cu_num_generated_tokens
    )
```
**EN:** This function implements `detach_zero_copy_from_model_runner_output` within the module. The docstring frames it as: Detach Ray SHM-channel zero-copy buffers from a ModelRunnerOutput in-place. Key calls include `_copy_if_readonly`, `type`, `isinstance`, `copy`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `detach_zero_copy_from_model_runner_output`，其作用域位于the module。 关键调用包括 `_copy_if_readonly`, `type`, `isinstance`, `copy`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FutureWrapper` class / `FutureWrapper` 类
```python
class FutureWrapper(Future):
    """A wrapper around Ray output reference to meet the interface
    of .execute_model(): The top level (core busy loop) expects .result() api
    to block and return a single output.

    If aggregator is provided, the outputs from all workers are aggregated upon
    the result() call. If not only the first worker's output is returned.
    """
```
**EN:** Introduces the `FutureWrapper` class on top of `Future`. Core methods include `__init__`, `result`. Docstring signal: A wrapper around Ray output reference to meet the interface of .execute_model(): The top level (core busy loop) expects .result() api to block and return a single output.
**CN:** 这里定义 `FutureWrapper` 类，其基类包括 `Future`。核心方法包括 `__init__`, `result`。

### `FutureWrapper.__init__` method / `FutureWrapper.__init__` 方法
```python
    def __init__(self, ref_or_refs, aggregator: KVOutputAggregator | None = None):
        super().__init__()
        self.ref_or_refs = ref_or_refs
        self.aggregator = aggregator
```
**EN:** This method initializes the object state within `FutureWrapper`. Key calls include `__init__`, `super`. It touches state such as `ref_or_refs`, `aggregator`.
**CN:** 该方法会初始化对象状态，其作用域位于`FutureWrapper`。 关键调用包括 `__init__`, `super`。 它会读写 `ref_or_refs`, `aggregator` 等状态。

### `FutureWrapper.result` method / `FutureWrapper.result` 方法
```python
    def result(self, timeout=None):
        outputs = ray.get(self.ref_or_refs, timeout=timeout)
        if self.aggregator is None:
            detach_zero_copy_from_model_runner_output(outputs)
            return outputs

        for output in outputs:
            detach_zero_copy_from_model_runner_output(output)
        return self.aggregator.aggregate(outputs, output_rank=0)
```
**EN:** This method implements `result` within `FutureWrapper`. Key calls include `get`, `aggregate`, `detach_zero_copy_from_model_runner_output`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `result`，其作用域位于`FutureWrapper`。 关键调用包括 `get`, `aggregate`, `detach_zero_copy_from_model_runner_output`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `assert_ray_available` function / `assert_ray_available` 函数
```python
def assert_ray_available():
    """Raise an exception if Ray is not available."""
    if ray is None:
        raise ValueError(
            f"Failed to import Ray: {ray_import_err}."
            "Please install Ray with `pip install ray`."
        )
```
**EN:** This function implements `assert_ray_available` within the module. The docstring frames it as: Raise an exception if Ray is not available. Key calls include `ValueError`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `assert_ray_available`，其作用域位于the module。 关键调用包括 `ValueError`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_verify_bundles` function / `_verify_bundles` 函数
```python
def _verify_bundles(
    placement_group: "PlacementGroup",
    parallel_config: ParallelConfig,
    device_str: str,
    require_gpu_on_driver: bool = True,
):
    """Verify a given placement group has bundles located in the right place.

    There are 2 rules.
    - Warn if all tensor parallel workers cannot fit in a single node.
    - Fail if driver node is not included in a placement group
      (only when require_gpu_on_driver is True).
    """
    assert ray.is_initialized(), (
        "Ray is not initialized although distributed-executor-backend is ray."
    )
    pg_data = placement_group_table(placement_group)
    # bundle_idx -> node_id
    bundle_to_node_ids = pg_data["bundles_to_node_id"]
    # bundle_idx -> bundle (e.g., {"GPU": 1})
    bundles = pg_data["bundles"]
    # node_id -> List of bundle (e.g., {"GPU": 1})
    node_id_to_bundle: dict[str, list[dict[str, float]]] = defaultdict(list)

    for bundle_idx, node_id in bundle_to_node_ids.items():
        node_id_to_bundle[node_id].append(bundles[bundle_idx])
    driver_node_id = ray.get_runtime_context().get_node_id()

    if require_gpu_on_driver and driver_node_id not in node_id_to_bundle:
        raise RuntimeError(
            f"driver node id {driver_node_id} is not included in a placement "
            f"group {placement_group.id}. Node id -> bundles "
            f"{node_id_to_bundle}. "
            "You don't have enough GPUs available in a current node. Check "
            "`ray status` and `ray list nodes` to see if you have available "
            "GPUs in a node `{driver_node_id}` before starting an vLLM engine."
        )

    for node_id, bundles in node_id_to_bundle.items():
        if len(bundles) < parallel_config.tensor_parallel_size:
            logger.warning(
                "tensor_parallel_size=%d "
                "is bigger than a reserved number of %ss (%d "
                "%ss) in a node %s. Tensor parallel workers can be "
                "spread out to 2+ nodes which can degrade the performance "
                "unless you have fast interconnect across nodes, like "
                "Infiniband. To resolve this issue, make sure you have more "
                "than %d GPUs available at each node.",
                parallel_config.tensor_parallel_size,
                device_str,
                len(bundles),
                device_str,
                node_id,
                parallel_config.tensor_parallel_size,
            )
```
**EN:** This function implements `_verify_bundles` within the module. The docstring frames it as: Verify a given placement group has bundles located in the right place. Key calls include `is_initialized`, `placement_group_table`, `defaultdict`, `items`, `get_node_id`, `append`. The control flow contains 2 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_verify_bundles`，其作用域位于the module。 关键调用包括 `is_initialized`, `placement_group_table`, `defaultdict`, `items`, `get_node_id`, `append`。 控制流包含 2 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `build_actor_name` function / `build_actor_name` 函数
```python
def build_actor_name(
    instance_id: str,
    rank: int,
    tp_size: int,
    pp_size: int,
    pcp_size: int,
) -> str:
    """Build a descriptive Ray actor name for dashboard visibility."""
    name = f"vllm_Worker_{instance_id}"
    if tp_size > 1:
        name += f"_TP{rank % tp_size}"
    if pp_size > 1:
        name += f"_PP{(rank // tp_size) % pp_size}"
    if pcp_size > 1:
        name += f"_PCP{rank // (tp_size * pp_size)}"
    return name
```
**EN:** This function builds derived structures within the module. The docstring frames it as: Build a descriptive Ray actor name for dashboard visibility. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会构建派生结构，其作用域位于the module。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `get_bundles_for_indices` function / `get_bundles_for_indices` 函数
```python
def get_bundles_for_indices(
    placement_group: "PlacementGroup",
    bundle_indices: list[int],
    world_size: int,
) -> list[tuple[int, str, str]]:
    """
    Return GPU bundle indices paired with node IDs and node IPs for
    explicit bundle indices specified via VLLM_RAY_BUNDLE_INDICES.
    """
    assert len(bundle_indices) == world_size, (
        "VLLM_RAY_BUNDLE_INDICES must have the same size"
        f" as the world size, but got {bundle_indices=} "
        f"and {world_size=}"
    )
    assert len(set(bundle_indices)) == len(bundle_indices), (
        "VLLM_RAY_BUNDLE_INDICES cannot have duplicate values,"
        f" but got {bundle_indices=}"
    )

    pg_data = placement_group_table(placement_group)
    pg_bundle_to_node = pg_data["bundles_to_node_id"]
    node_id_to_ip = {
        n["NodeID"]: n["NodeManagerAddress"] for n in ray.nodes() if n["Alive"]
    }
    return [
        (bid, pg_bundle_to_node[bid], node_id_to_ip[pg_bundle_to_node[bid]])
        for bid in bundle_indices
    ]
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Return GPU bundle indices paired with node IDs and node IPs for explicit bundle indices specified via VLLM_RAY_BUNDLE_INDICES. Key calls include `placement_group_table`, `len`, `set`, `nodes`.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `placement_group_table`, `len`, `set`, `nodes`。

### `get_bundles_sorted_by_node` function / `get_bundles_sorted_by_node` 函数
```python
def get_bundles_sorted_by_node(
    placement_group: "PlacementGroup",
) -> list[tuple[int, str, str]]:
    """
    Return GPU bundle indices paired with node IDs and node IPs,
    sorted driver-first.

    This utility has to be invoked from the driver node.

    Example: 3-node cluster, driver on node-A, PG bundles spread
    across nodes:

      Input: [
          (0, node-C),
          (1, node-A),
          (2, node-B),
          (3, node-C),
          (4, node-A),
          (5, node-B),
      ]
      Output: [
          (1, node-A),
          (4, node-A),
          (2, node-B),
          (5, node-B),
          (0, node-C),
          (3, node-C),
      ]
    """
    pg_data = placement_group_table(placement_group)
    bundle_to_node = pg_data["bundles_to_node_id"]

    ray_device_key = current_platform.ray_device_key
    if not ray_device_key:
        raise ValueError(
            f"current platform {current_platform.device_name} does not support ray."
        )

    node_id_to_ip = {
        n["NodeID"]: n["NodeManagerAddress"] for n in ray.nodes() if n["Alive"]
    }

    bundle_specs = placement_group.bundle_specs
    assert bundle_specs is not None
    bundle_to_node_id: list[tuple[int, str, str]] = []
    for bundle_idx, bundle in enumerate(bundle_specs):
        if bundle.get(ray_device_key):
            node_id = bundle_to_node.get(bundle_idx)
            bundle_to_node_id.append((bundle_idx, node_id, node_id_to_ip[node_id]))

    driver_node = ray.get_runtime_context().get_node_id()

    def _sort_key(item):
        _, node_id, _ = item
        return (0 if node_id == driver_node else 1, node_id)

    bundle_to_node_id.sort(key=_sort_key)

    return bundle_to_node_id
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Return GPU bundle indices paired with node IDs and node IPs, sorted driver-first. Key calls include `placement_group_table`, `enumerate`, `get_node_id`, `sort`, `ValueError`, `get`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `placement_group_table`, `enumerate`, `get_node_id`, `sort`, `ValueError`, `get`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `_wait_until_pg_ready` function / `_wait_until_pg_ready` 函数
```python
def _wait_until_pg_ready(current_placement_group: "PlacementGroup"):
    """Wait until a placement group is ready.

    It prints the informative log messages if the placement group is
    not created within time.

    """
    # Wait until PG is ready - this will block until all
    # requested resources are available, and will time out
    # if they cannot be provisioned.
    placement_group_specs = current_placement_group.bundle_specs

    s = time.time()
    pg_ready_ref = current_placement_group.ready()
    wait_interval = 10
    while time.time() - s < PG_WAIT_TIMEOUT:
        ready, _ = ray.wait([pg_ready_ref], timeout=wait_interval)
        if len(ready) > 0:
            break

        # Exponential backoff for warning print.
        wait_interval *= 2
        logger.info(
            "Waiting for creating a placement group of specs for "
            "%d seconds. specs=%s. Check `ray status` and "
            "`ray list nodes` to see if you have enough resources,"
            " and make sure the IP addresses used by ray cluster"
            " are the same as VLLM_HOST_IP environment variable"
            " specified in each node if you are running on a multi-node.",
            int(time.time() - s),
            placement_group_specs,
        )

    try:
        ray.get(pg_ready_ref, timeout=0)
    except ray.exceptions.GetTimeoutError:
        # Provide more helpful error message when GPU count is exceeded
        total_gpu_required = sum(spec.get("GPU", 0) for spec in placement_group_specs)
        # If more than one GPU is required for the placement group, provide a
        # more specific error message.
        # We use >1 here because multi-GPU (tensor parallel) jobs are more
        # likely to fail due to insufficient cluster resources, and users may
        # need to adjust tensor_parallel_size to fit available GPUs.
        if total_gpu_required > 1:
            raise ValueError(
                f"Cannot provide a placement group requiring "
                f"{total_gpu_required} GPUs "
                f"(placement_group_specs={placement_group_specs}) within "
                f"{PG_WAIT_TIMEOUT} seconds.\n"
                f"Tensor parallel size may exceed available GPUs in your "
                f"cluster. Check resources with `ray status` and "
                f"`ray list nodes`.\n"
                f"If running on K8s with limited GPUs, consider reducing "
                f"--tensor-parallel-size to match available GPU resources."
            ) from None
        else:
            raise ValueError(
                "Cannot provide a placement group of "
                f"{placement_group_specs=} within "
                f"{PG_WAIT_TIMEOUT} seconds. See "
                "`ray status` and `ray list nodes` to make sure the cluster "
                "has enough resources."
            ) from None
```
**EN:** This function implements `_wait_until_pg_ready` within the module. The docstring frames it as: Wait until a placement group is ready. Key calls include `time`, `ready`, `wait`, `info`, `get`, `len`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_wait_until_pg_ready`，其作用域位于the module。 关键调用包括 `time`, `ready`, `wait`, `info`, `get`, `len`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `initialize_ray_cluster` function / `initialize_ray_cluster` 函数
```python
def initialize_ray_cluster(
    parallel_config: ParallelConfig,
    ray_address: str | None = None,
    require_gpu_on_driver: bool = True,
):
    """Initialize the distributed cluster with Ray.

    it will connect to the Ray cluster and create a placement group
    for the workers, which includes the specification of the resources
    for each distributed worker.

    Args:
        parallel_config: The configurations for parallel execution.
        ray_address: The address of the Ray cluster. If None, uses
            the default Ray cluster address.
        require_gpu_on_driver: If True (default), require at least one GPU
            on the current (driver) node and pin the first PG bundle to it.
            Set to False for executors like RayExecutorV2 where all GPU work
            is delegated to remote Ray actors.
    """
    assert_ray_available()
    from vllm.platforms import current_platform

    # Disable Ray usage stats collection
    if os.environ.get("RAY_USAGE_STATS_ENABLED", "0") != "1":
        os.environ["RAY_USAGE_STATS_ENABLED"] = "0"

    # Prevalidate GPU requirements before Ray processing
    if current_platform.is_cuda() and parallel_config.world_size > 1:
        available_gpus = current_platform.device_count()
        if parallel_config.world_size > available_gpus:
            logger.warning(
                "Tensor parallel size (%d) exceeds available GPUs (%d). "
                "This may result in Ray placement group allocation failures. "
                "Consider reducing tensor_parallel_size to %d or less, "
                "or ensure your Ray cluster has %d GPUs available.",
                parallel_config.world_size,
                available_gpus,
                available_gpus,
                parallel_config.world_size,
            )

    if ray.is_initialized():
        logger.info("Ray is already initialized. Skipping Ray initialization.")
    elif current_platform.is_rocm() or current_platform.is_xpu():
        # Try to connect existing ray instance and create a new one if not found
        try:
            ray.init("auto")
        except ConnectionError:
            logger.warning(
# ... omitted for brevity ...
        if require_gpu_on_driver:
            if current_node_resource.get(device_str, 0) < 1:
                raise ValueError(
                    f"Current node has no {device_str} available. "
                    f"{current_node_resource=}. vLLM engine cannot start "
                    f"without {device_str}. Make sure you have at least 1 "
                    f"{device_str} available in a node "
                    f"{current_node_id=} {current_ip=}."
                )
            # This way, at least bundle is required to be created in a
            # current node.
            placement_group_specs[0][f"node:{current_ip}"] = 0.001

        # By default, Ray packs resources as much as possible.
        current_placement_group = ray.util.placement_group(
            placement_group_specs, strategy="PACK"
        )
        _wait_until_pg_ready(current_placement_group)

    assert current_placement_group is not None
    _verify_bundles(
        current_placement_group, parallel_config, device_str, require_gpu_on_driver
    )
    # Set the placement group in the parallel config
    parallel_config.placement_group = current_placement_group
```
**EN:** This function implements `initialize_ray_cluster` within the module. The docstring frames it as: Initialize the distributed cluster with Ray. Key calls include `assert_ray_available`, `is_initialized`, `_verify_bundles`, `get`, `is_cuda`, `device_count`. The control flow contains 15 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `initialize_ray_cluster`，其作用域位于the module。 关键调用包括 `assert_ray_available`, `is_initialized`, `_verify_bundles`, `get`, `is_cuda`, `device_count`。 控制流包含 15 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `get_num_tpu_nodes` function / `get_num_tpu_nodes` 函数
```python
def get_num_tpu_nodes() -> int:
    from ray._private.accelerators import TPUAcceleratorManager

    cluster_resources = ray.cluster_resources()
    total_tpus = int(cluster_resources["TPU"])
    tpus_per_node = TPUAcceleratorManager.get_current_node_num_accelerators()
    assert total_tpus % tpus_per_node == 0
    return total_tpus // tpus_per_node
```
**EN:** This function returns or derives a value within the module. Key calls include `cluster_resources`, `int`, `get_current_node_num_accelerators`.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `cluster_resources`, `int`, `get_current_node_num_accelerators`。

### `get_num_nodes_in_placement_group` function / `get_num_nodes_in_placement_group` 函数
```python
def get_num_nodes_in_placement_group() -> int:
    pg_table = ray.util.placement_group_table()
    current_pg = ray.util.get_current_placement_group()
    num_nodes = 0

    if current_pg:
        nodes_in_pg = set()
        for pg_key, pg in pg_table.items():
            if pg_key == current_pg.id.hex():
                for _, node in pg["bundles_to_node_id"].items():
                    nodes_in_pg.add(node)
        num_nodes = len(nodes_in_pg)

    return num_nodes
```
**EN:** This function returns or derives a value within the module. Key calls include `placement_group_table`, `get_current_placement_group`, `set`, `items`, `len`, `hex`. The control flow contains 2 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `placement_group_table`, `get_current_placement_group`, `set`, `items`, `len`, `hex`。 控制流包含 2 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `detach_zero_copy_from_model_runner_output`: top-level helper or orchestration entry point. / `detach_zero_copy_from_model_runner_output`：顶层辅助函数或编排入口。
- `FutureWrapper`: central class or interface in this module. / `FutureWrapper`：本模块中的核心类或接口。
- `ray_is_available`: top-level helper or orchestration entry point. / `ray_is_available`：顶层辅助函数或编排入口。
- `assert_ray_available`: top-level helper or orchestration entry point. / `assert_ray_available`：顶层辅助函数或编排入口。
- `_verify_bundles`: top-level helper or orchestration entry point. / `_verify_bundles`：顶层辅助函数或编排入口。
- `build_actor_name`: top-level helper or orchestration entry point. / `build_actor_name`：顶层辅助函数或编排入口。
- `get_bundles_for_indices`: top-level helper or orchestration entry point. / `get_bundles_for_indices`：顶层辅助函数或编排入口。
- `get_bundles_sorted_by_node`: top-level helper or orchestration entry point. / `get_bundles_sorted_by_node`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `os`, `time`, `collections`, `concurrent`, `typing`
- External / 外部依赖: `numpy`, `ray`
- Internal vLLM / 内部依赖: `vllm.platforms`, `vllm.config`, `vllm.distributed`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.logger`, `vllm.sequence`, `vllm.utils.network_utils`, `vllm.v1.outputs`, `vllm.v1.serial_utils`, `vllm.v1.worker.worker_base`, `vllm.v1.core.sched.output`

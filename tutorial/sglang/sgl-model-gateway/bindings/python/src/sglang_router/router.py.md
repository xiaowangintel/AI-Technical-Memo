# router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/bindings/python/src/sglang_router/router.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements classes `Router` and functions `policy_from_str`, `backend_from_str`, `history_backend_from_str`, `role_from_str`, and 1 more for the Python router binding. Other gateway components import it to reuse configuration, control flow, or shared utilities. / 该模块为 Python 路由绑定层 实现了 类 `Router`与函数 `policy_from_str`, `backend_from_str`, `history_backend_from_str`, `role_from_str`, and 1 more，其他网关组件会导入它以复用配置、控制流程或共享工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Imports and dependencies
```python
import logging
from typing import Optional

from sglang_router.router_args import RouterArgs
```
**EN:** This block imports `logging`, `typing`, `sglang_router.router_args`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 6-6: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 7-19: Imports and dependencies
```python
from sglang_router.sglang_router_rs import (
    BackendType,
    HistoryBackendType,
    PolicyType,
    PyApiKeyEntry,
    PyControlPlaneAuthConfig,
    PyJwtConfig,
    PyOracleConfig,
    PyPostgresConfig,
    PyRedisConfig,
    PyRole,
)
from sglang_router.sglang_router_rs import Router as _Router
```
**EN:** This block imports `sglang_router.sglang_router_rs`, `sglang_router.sglang_router_rs`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 22-36: function `policy_from_str`
```python
def policy_from_str(policy_str: Optional[str]) -> PolicyType:
    """Convert policy string to PolicyType enum."""
    if policy_str is None:
        return None
    policy_map = {
        "random": PolicyType.Random,
        "round_robin": PolicyType.RoundRobin,
        "cache_aware": PolicyType.CacheAware,
        "power_of_two": PolicyType.PowerOfTwo,
        "bucket": PolicyType.Bucket,
        "manual": PolicyType.Manual,
        "consistent_hashing": PolicyType.ConsistentHashing,
        "prefix_hash": PolicyType.PrefixHash,
    }
    return policy_map[policy_str]
```
**EN:** function `policy_from_str` is described as: Convert policy string to PolicyType enum.
**CN:** 函数 `policy_from_str` 封装了与 `policy from str` 相关的逻辑，供本模块中的调用方复用。

### Lines 39-51: function `backend_from_str`
```python
def backend_from_str(backend_str: Optional[str]) -> BackendType:
    """Convert backend string to BackendType enum."""
    if isinstance(backend_str, BackendType):
        return backend_str
    if backend_str is None:
        return BackendType.Sglang
    backend_map = {"sglang": BackendType.Sglang, "openai": BackendType.Openai}
    backend_lower = backend_str.lower()
    if backend_lower not in backend_map:
        raise ValueError(
            f"Unknown backend: {backend_str}. Valid options: {', '.join(backend_map.keys())}"
        )
    return backend_map[backend_lower]
```
**EN:** function `backend_from_str` is described as: Convert backend string to BackendType enum.
**CN:** 函数 `backend_from_str` 封装了与 `backend from str` 相关的逻辑，供本模块中的调用方复用。

### Lines 54-73: function `history_backend_from_str`
```python
def history_backend_from_str(backend_str: Optional[str]) -> HistoryBackendType:
    """Convert history backend string to HistoryBackendType enum."""
    if isinstance(backend_str, HistoryBackendType):
        return backend_str
    if backend_str is None:
        return HistoryBackendType.Memory
    backend_lower = backend_str.lower()
    if backend_lower == "memory":
        return HistoryBackendType.Memory
    elif backend_lower == "none":
        # Use getattr to access 'None' which is a Python keyword
        return getattr(HistoryBackendType, "None")
    elif backend_lower == "oracle":
        return HistoryBackendType.Oracle
    elif backend_lower == "postgres":
        return HistoryBackendType.Postgres
    elif backend_lower == "redis":
        return HistoryBackendType.Redis
    else:
        raise ValueError(f"Unknown history backend: {backend_str}")
```
**EN:** function `history_backend_from_str` is described as: Convert history backend string to HistoryBackendType enum.
**CN:** 函数 `history_backend_from_str` 封装了与 `history backend from str` 相关的逻辑，供本模块中的调用方复用。

### Lines 76-80: function `role_from_str`
```python
def role_from_str(role_str: str) -> PyRole:
    """Convert role string to PyRole enum."""
    if role_str.lower() == "admin":
        return PyRole.Admin
    return PyRole.User
```
**EN:** function `role_from_str` is described as: Convert role string to PyRole enum.
**CN:** 函数 `role_from_str` 封装了与 `role from str` 相关的逻辑，供本模块中的调用方复用。

### Lines 83-144: function `build_control_plane_auth_config`
```python
def build_control_plane_auth_config(
    args_dict: dict,
) -> Optional[PyControlPlaneAuthConfig]:
    """Build control plane auth config from args dict."""
    api_keys = args_dict.get("control_plane_api_keys", [])
    jwt_issuer = args_dict.get("jwt_issuer")
    jwt_audience = args_dict.get("jwt_audience")
    audit_enabled = args_dict.get("control_plane_audit_enabled", False)

    # Check if any auth is configured
    has_api_keys = bool(api_keys)
    has_jwt = jwt_issuer is not None and jwt_audience is not None

    # Warn when JWT-related fields are set but the required pair is missing —
    # otherwise the silent drop here disables JWT auth without explanation.
    jwt_partial = bool(
        args_dict.get("jwt_jwks_uri")
        or args_dict.get("jwt_role_mapping")
        or (
            args_dict.get("jwt_role_claim") is not None
            and args_dict.get("jwt_role_claim") != "roles"
        )
    )
    if jwt_partial and not has_jwt:
        logger.warning(
            "JWT-related fields set but jwt_issuer/jwt_audience missing; "
            "JWT auth will NOT be enabled."
        )

    if not has_api_keys and not has_jwt:
        return None

    # Build API key entries
    py_api_keys = []
    for key_tuple in api_keys:
        # Tuple format: (id, name, key, role)
        key_id, name, key, role = key_tuple
        py_api_keys.append(
            PyApiKeyEntry(
                id=key_id,
                name=name,
                key=key,
                role=role_from_str(role),
            )
        )

    # Build JWT config if present
    jwt_config = None
    if has_jwt:
        jwt_config = PyJwtConfig(
            issuer=jwt_issuer,
            audience=jwt_audience,
            jwks_uri=args_dict.get("jwt_jwks_uri"),
            role_claim=args_dict.get("jwt_role_claim", "roles"),
            role_mapping=args_dict.get("jwt_role_mapping", {}),
        )

    return PyControlPlaneAuthConfig(
        jwt=jwt_config,
        api_keys=py_api_keys,
        audit_enabled=audit_enabled,
    )
```
**EN:** function `build_control_plane_auth_config` is described as: Build control plane auth config from args dict.
**CN:** 函数 `build_control_plane_auth_config` 封装了与 `build control plane auth config` 相关的逻辑，供本模块中的调用方复用。

### Lines 147-341: Class `Router`
```python
class Router:
    """
    A high-performance router for distributing requests across worker nodes.

    Args:
        worker_urls: List of URLs for worker nodes that will handle requests. Each URL should include
            the protocol, host, and port (e.g., ['http://worker1:8000', 'http://worker2:8000'])
        policy: Load balancing policy to use. Options:
            - PolicyType.Random: Randomly select workers
            - PolicyType.RoundRobin: Distribute requests in round-robin fashion
            - PolicyType.CacheAware: Distribute requests based on cache state and load balance
            - PolicyType.PowerOfTwo: Select best of two random workers based on load (PD mode only)
        host: Host address to bind the router server. Supports IPv4, IPv6 (e.g., ::, ::1), or 0.0.0.0 for all interfaces. Default: '0.0.0.0'
        port: Port number to bind the router server. Default: 3001
        worker_startup_timeout_secs: Timeout in seconds for worker startup and registration. Large models can take significant time to load into GPU memory. Default: 1800 (30 minutes)
        worker_startup_check_interval: Interval in seconds between checks for worker initialization. Default: 10
        cache_threshold: Cache threshold (0.0-1.0) for cache-aware routing. Routes to cached worker
            if the match rate exceeds threshold, otherwise routes to the worker with the smallest
            tree. Default: 0.5
        balance_abs_threshold: Load balancing is triggered when (max_load - min_load) > abs_threshold
            AND max_load > min_load * rel_threshold. Otherwise, use cache aware. Default: 32
        balance_rel_threshold: Load balancing is triggered when (max_load - min_load) > abs_threshold
            AND max_load > min_load * rel_threshold. Otherwise, use cache aware. Default: 1.0001
        eviction_interval_secs: Interval in seconds between cache eviction operations in cache-aware
            routing. Default: 60
        max_payload_size: Maximum payload size in bytes. Default: 256MB
        max_tree_size: Maximum size of the approximation tree for cache-aware routing. Default: 2^24
        dp_aware: Enable data parallelism aware schedule. Default: False
        enable_igw: Enable IGW (Inference-Gateway) mode for multi-model support. When enabled,
            the router can manage multiple models simultaneously with per-model load balancing
            policies. Default: False
        api_key: The api key used for the authorization with the worker.
            Useful when the dp aware scheduling strategy is enabled.
            Default: None
        log_dir: Directory to store log files. If None, logs are only output to console. Default: None
        log_level: Logging level. Options: 'debug', 'info', 'warn', 'error'.
        service_discovery: Enable Kubernetes service discovery. When enabled, the router will
            automatically discover worker pods based on the selector. Default: False
        selector: Dictionary mapping of label keys to values for Kubernetes pod selection.
            Example: {"app": "sglang-worker"}. Default: {}
        service_discovery_port: Port to use for service discovery. The router will generate
            worker URLs using this port. Default: 80
        service_discovery_namespace: Kubernetes namespace to watch for pods. If not provided,
            watches pods across all namespaces (requires cluster-wide permissions). Default: None
        prefill_selector: Dictionary mapping of label keys to values for Kubernetes pod selection
            for prefill servers (PD mode only). Default: {}
        decode_selector: Dictionary mapping of label keys to values for Kubernetes pod selection
            for decode servers (PD mode only). Default: {}
        prometheus_port: Port to expose Prometheus metrics. Default: None
        prometheus_host: Host address to bind the Prometheus metrics server. Default: None
        pd_disaggregation: Enable PD (Prefill-Decode) disaggregated mode. Default: False
        prefill_urls: List of (url, bootstrap_port) tuples for prefill servers (PD mode only)
        decode_urls: List of URLs for decode servers (PD mode only)
        prefill_policy: Specific load balancing policy for prefill nodes (PD mode only).
            If not specified, uses the main policy. Default: None
        decode_policy: Specific load balancing policy for decode nodes (PD mode only).
            If not specified, uses the main policy. Default: None
        request_id_headers: List of HTTP headers to check for request IDs. If not specified,
            uses common defaults: ['x-request-id', 'x-correlation-id', 'x-trace-id', 'request-id'].
            Example: ['x-my-request-id', 'x-custom-trace-id']. Default: None
        bootstrap_port_annotation: Kubernetes annotation name for bootstrap port (PD mode).
            Default: 'sglang.ai/bootstrap-port'
        request_timeout_secs: Request timeout in seconds. Default: 600
        max_concurrent_requests: Maximum number of concurrent requests allowed for rate limiting. Default: 256
        queue_size: Queue size for pending requests when max concurrent limit reached (0 = no queue, return 429 immediately). Default: 100
        queue_timeout_secs: Maximum time (in seconds) a request can wait in queue before timing out. Default: 60
        rate_limit_tokens_per_second: Token bucket refill rate (tokens per second). If not set, defaults to max_concurrent_requests. Default: None
        cors_allowed_origins: List of allowed origins for CORS. Empty list allows all origins. Default: []
        health_failure_threshold: Number of consecutive health check failures before marking worker unhealthy. Default: 3
        health_success_threshold: Number of consecutive health check successes before marking worker healthy. Default: 2
        health_check_timeout_secs: Timeout in seconds for health check requests. Default: 5
        health_check_interval_secs: Interval in seconds between runtime health checks. Default: 60
        health_check_endpoint: Health check endpoint path. Default: '/health'
        model_path: Model path for loading tokenizer (HuggingFace model ID or local path). Default: None
        tokenizer_path: Explicit tokenizer path (overrides model_path tokenizer if provided). Default: None
        server_cert_path: Path to server TLS certificate (PEM format). Default: None
        server_key_path: Path to server TLS private key (PEM format). Default: None
    """

    def __init__(self, router: _Router):
        self._router = router

    @staticmethod
    def from_args(args: RouterArgs) -> "Router":
        """Create a router from a RouterArgs instance."""

        args_dict = vars(args)
        # Convert RouterArgs to _Router parameters
        args_dict["worker_urls"] = (
            []
            if args_dict["service_discovery"] or args_dict["pd_disaggregation"]
            else args_dict["worker_urls"]
        )
        args_dict["policy"] = policy_from_str(args_dict["policy"])
        args_dict["prefill_urls"] = (
            args_dict["prefill_urls"] if args_dict["pd_disaggregation"] else None
        )
        args_dict["decode_urls"] = (
            args_dict["decode_urls"] if args_dict["pd_disaggregation"] else None
        )
        args_dict["prefill_policy"] = policy_from_str(args_dict["prefill_policy"])
        args_dict["decode_policy"] = policy_from_str(args_dict["decode_policy"])

        # Convert backend
        args_dict["backend"] = backend_from_str(args_dict.get("backend"))

        # Convert history_backend to enum first
        history_backend_raw = args_dict.get("history_backend", "memory")
        history_backend = history_backend_from_str(history_backend_raw)

        # Convert Oracle config if needed
        oracle_config = None
        if history_backend == HistoryBackendType.Oracle:
            # Prioritize TNS alias over connect descriptor
            tns_alias = args_dict.get("oracle_tns_alias")
            connect_descriptor = args_dict.get("oracle_connect_descriptor")

            # Use TNS alias if provided, otherwise use connect descriptor
            final_descriptor = tns_alias if tns_alias else connect_descriptor

            oracle_config = PyOracleConfig(
                password=args_dict.get("oracle_password"),
                username=args_dict.get("oracle_username"),
                connect_descriptor=final_descriptor,
                wallet_path=args_dict.get("oracle_wallet_path"),
                pool_min=args_dict.get("oracle_pool_min", 1),
                pool_max=args_dict.get("oracle_pool_max", 16),
                pool_timeout_secs=args_dict.get("oracle_pool_timeout_secs", 30),
            )
        args_dict["oracle_config"] = oracle_config
        args_dict["history_backend"] = history_backend

        # Convert Postgres config if needed
        postgres_config = None
        if history_backend == HistoryBackendType.Postgres:
            postgres_config = PyPostgresConfig(
                db_url=args_dict.get("postgres_db_url"),
                pool_max=args_dict.get("postgres_pool_max", 16),
            )
        args_dict["postgres_config"] = postgres_config

        # Convert Redis config if needed
        redis_config = None
        if history_backend == HistoryBackendType.Redis:
            retention_days = args_dict.get("redis_retention_days", 30)
            # If retention_days is negative, it means persistent storage (None in Rust)
            retention_arg = None if retention_days < 0 else retention_days

            redis_config = PyRedisConfig(
                url=args_dict.get("redis_url"),
                pool_max=args_dict.get("redis_pool_max", 16),
                retention_days=retention_arg,
            )
        args_dict["redis_config"] = redis_config

        # Build control plane auth config
        args_dict["control_plane_auth"] = build_control_plane_auth_config(args_dict)

        # Remove fields that shouldn't be passed to Rust Router constructor
        fields_to_remove = [
            "mini_lb",
            "test_external_dp_routing",
            "oracle_wallet_path",
            "oracle_tns_alias",
            "oracle_connect_descriptor",
            "oracle_username",
            "oracle_password",
            "oracle_pool_min",
            "oracle_pool_max",
            "oracle_pool_timeout_secs",
            "postgres_db_url",
            "postgres_pool_max",
            "redis_url",
            "redis_pool_max",
            "redis_retention_days",
            # Control plane auth fields (converted to control_plane_auth)
            "control_plane_api_keys",
            "control_plane_audit_enabled",
            "jwt_issuer",
            "jwt_audience",
            "jwt_jwks_uri",
            "jwt_role_claim",
            "jwt_role_mapping",
        ]
        for field in fields_to_remove:
            args_dict.pop(field, None)

        return Router(_Router(**args_dict))

    def start(self) -> None:
        """Start the router server.

        This method blocks until the server is shut down.
        """
        self._router.start()
```
**EN:** Class `Router` groups related state and behavior. It exposes 3 method(s) that implement the module's primary abstraction.
**CN:** 类 `Router` 将相关状态与行为封装在一起，提供 3 个方法来实现本模块的核心抽象。

## Key Concepts / 关键概念
- Routing, worker selection, or load balancing / 路由、工作节点选择或负载均衡
- Primary classes: `Router` / 主要类：`Router`
- Reusable functions: `policy_from_str`, `backend_from_str`, `history_backend_from_str`, `role_from_str`, and 1 more / 可复用函数：`policy_from_str`, `backend_from_str`, `history_backend_from_str`, `role_from_str`, and 1 more

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Third-party / 第三方**: None explicitly imported / 未显式导入
- **Internal / 内部模块**: `sglang_router.router_args`, `sglang_router.sglang_router_rs`

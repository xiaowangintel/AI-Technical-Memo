# disagg_args.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/disagg_args.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `DisaggArgsMixin`, `add_disagg_cli_args`, and `convert_disagg_role_string`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Disaggregated diffusion CLI arguments and helper methods. All disagg-related dataclass fields, argparse registration, and endpoint derivation logic live here. / 该文件属于解耦运行时层。它围绕 `DisaggArgsMixin`、`add_disagg_cli_args` 和 `convert_disagg_role_string` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-14: module setup and imports / 模块初始化与导入
```python
"""Disaggregated diffusion CLI arguments and helper methods.

All disagg-related dataclass fields, argparse registration, and endpoint
derivation logic live here.  ``ServerArgs`` inherits from
``DisaggArgsMixin`` so the fields appear on the top-level config object.
"""

from __future__ import annotations

import argparse
from typing import TYPE_CHECKING

from sglang.multimodal_gen.runtime.disaggregation.roles import RoleType
```
**EN:** This block establishes the module context and imports `__future__`, `argparse`, `typing`, and `sglang.multimodal_gen.runtime.disaggregation.roles`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`argparse`、`typing` 和 `sglang.multimodal_gen.runtime.disaggregation.roles`。这些依赖为后续实现提供所需符号。

### Lines 16-24: supporting statements / 辅助语句
```python
if TYPE_CHECKING:
    pass

# ── Port offsets for disagg result endpoints (deterministic convention) ──
DISAGG_RESULT_PORT_OFFSETS: dict[RoleType, int] = {
    RoleType.ENCODER: 1,
    RoleType.DENOISER: 2,
    RoleType.DECODER: 3,
}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `DISAGG_RESULT_PORT_OFFSETS`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `DISAGG_RESULT_PORT_OFFSETS` 等名称。

### Lines 27-34: `DisaggArgsMixin` class overview / `DisaggArgsMixin` 类概览
```python
class DisaggArgsMixin:
    """Methods for disaggregated diffusion, mixed into ``ServerArgs``.

    The dataclass **fields** remain in ``ServerArgs`` (to avoid MRO
    ordering issues with ``@dataclass`` inheritance).  This mixin only
    provides the methods that operate on those fields.
    """
```
**EN:** This block defines class `DisaggArgsMixin`. Methods for disaggregated diffusion, mixed into ``ServerArgs``. The dataclass **fields** remain in ``ServerArgs`` (to avoid MRO ordering issues with ``@dataclass`` inheritance).
**CN:** 该代码块定义了类 `DisaggArgsMixin`。 它用于封装 disagg args mixin 相关行为。

### Lines 35-59: `get_role_parallelism` implementation / `get_role_parallelism` 实现
```python
    def get_role_parallelism(self, role_type: RoleType) -> dict[str, int | None]:
        """Return per-role parallelism overrides for the given role.

        Returns a dict with keys tp_size, sp_degree, ulysses_degree,
        ring_degree.  Values are ``None`` when not explicitly set
        (auto-derive from ``num_gpus``).
        """
        _none: dict[str, int | None] = {
            "tp_size": None,
            "sp_degree": None,
            "ulysses_degree": None,
            "ring_degree": None,
        }
        if role_type == RoleType.ENCODER:
            return {**_none, "tp_size": self.encoder_tp}
        elif role_type == RoleType.DENOISER:
            return {
                "tp_size": self.denoiser_tp,
                "sp_degree": self.denoiser_sp,
                "ulysses_degree": self.denoiser_ulysses,
                "ring_degree": self.denoiser_ring,
            }
        elif role_type == RoleType.DECODER:
            return {**_none, "tp_size": self.decoder_tp}
        return _none
```
**EN:** This block defines method `get_role_parallelism` on `DisaggArgsMixin`. Return per-role parallelism overrides for the given role. Returns a dict with keys tp_size, sp_degree, ulysses_degree, ring_degree. The implementation branches on conditions. Parameters such as `role_type` drive the behavior in this section.
**CN:** 该代码块定义了 `DisaggArgsMixin` 的方法 `get_role_parallelism`。 它用于获取role parallelism。 实现中包含条件分支。 本段逻辑主要由 `role_type` 等参数驱动。

### Lines 61-75: `derive_pool_result_endpoint` implementation / `derive_pool_result_endpoint` 实现
```python
    def derive_pool_result_endpoint(self) -> str:
        """Derive the result PUSH endpoint from ``disagg_server_addr`` + role.

        Convention: DS binds result PULL on ``scheduler_port + {1,2,3}``
        for encoder / denoiser / decoder.
        """
        if self.disagg_server_addr is None:
            raise ValueError("disagg_server_addr is required for per-role launch")
        addr = self.disagg_server_addr
        if addr.startswith("tcp://"):
            addr = addr[len("tcp://") :]
        host, port_str = addr.rsplit(":", 1)
        base_port = int(port_str)
        offset = DISAGG_RESULT_PORT_OFFSETS[self.disagg_role]
        return f"tcp://{host}:{base_port + offset}"
```
**EN:** This block defines method `derive_pool_result_endpoint` on `DisaggArgsMixin`. Derive the result PUSH endpoint from ``disagg_server_addr`` + role. Convention: DS binds result PULL on ``scheduler_port + {1,2,3}`` for encoder / denoiser / decoder. Key calls include `addr.startswith`, `addr.rsplit`, `int`, `ValueError`, and `len`. The implementation branches on conditions.
**CN:** 该代码块定义了 `DisaggArgsMixin` 的方法 `derive_pool_result_endpoint`。 它用于处理 derive pool result endpoint 相关逻辑。 关键调用包括 `addr.startswith`、`addr.rsplit`、`int`、`ValueError` 和 `len`。 实现中包含条件分支。

### Lines 77-79: `derive_pool_work_endpoint` implementation / `derive_pool_work_endpoint` 实现
```python
    def derive_pool_work_endpoint(self) -> str:
        """Derive the work PULL bind endpoint for a standalone role instance."""
        return f"tcp://0.0.0.0:{self.scheduler_port}"
```
**EN:** This block defines method `derive_pool_work_endpoint` on `DisaggArgsMixin`. Derive the work PULL bind endpoint for a standalone role instance.
**CN:** 该代码块定义了 `DisaggArgsMixin` 的方法 `derive_pool_work_endpoint`。 它用于处理 derive pool work endpoint 相关逻辑。

### Lines 85-187: `add_disagg_cli_args` implementation / `add_disagg_cli_args` 实现
```python
def add_disagg_cli_args(parser: argparse.ArgumentParser) -> None:
    """Register all disaggregated-diffusion CLI arguments as a group."""

    g = parser.add_argument_group(
        "Disaggregated diffusion",
        "Split the pipeline into independent Encoder / Denoiser / Decoder "
        "roles, each on its own GPU(s).  A DiffusionServer head node routes "
        "requests.  See docs/disaggregation.md for details.",
    )

    # Core
    g.add_argument(
        "--base-gpu-id",
        type=int,
        default=0,
        help="Starting GPU ID for this instance.  Used with --disagg-role "
        "to place role instances on specific GPUs without CUDA_VISIBLE_DEVICES.",
    )
    g.add_argument(
        "--disagg-role",
        type=str,
        default=RoleType.MONOLITHIC.value,
        choices=RoleType.choices(),
        help="Role for disaggregated pipeline.  "
        "'monolithic' (default): single server.  "
        "'encoder' / 'denoiser' / 'decoder': role instance.  "
        "'server': DiffusionServer head node (no GPU).  "
        "Role instances require --disagg-server-addr.  "
        "Server requires --encoder-urls, --denoiser-urls, --decoder-urls.",
    )
    g.add_argument(
        "--disagg-server-addr",
        type=str,
        default=None,
        help="DiffusionServer head node address (tcp://HOST:PORT).  "
        "Required for role instances.",
    )
    g.add_argument(
        "--disagg-timeout",
        type=int,
        default=600,
        help="Timeout in seconds for pending disagg requests (default: 600).",
    )
    g.add_argument(
        "--disagg-dispatch-policy",
        type=str,
        default="round_robin",
        choices=["round_robin", "max_free_slots"],
        help="Dispatch policy: 'round_robin' or 'max_free_slots' (default: round_robin).",
    )

    # Server head: remote instance URLs
    g.add_argument(
        "--encoder-urls",
        type=str,
        default=None,
        help="Encoder work endpoints (semicolon-separated).  "
        "Example: 'tcp://10.0.0.1:35000;tcp://10.0.0.2:35000'.",
    )
    g.add_argument(
        "--denoiser-urls",
        type=str,
        default=None,
        help="Denoiser work endpoints (semicolon-separated).",
    )
    g.add_argument(
        "--decoder-urls",
        type=str,
        default=None,
        help="Decoder work endpoints (semicolon-separated).",
    )

    # Per-role parallelism
    g.add_argument("--encoder-tp", type=int, default=None, help="Encoder TP degree.")
    g.add_argument("--denoiser-tp", type=int, default=None, help="Denoiser TP degree.")
    g.add_argument("--denoiser-sp", type=int, default=None, help="Denoiser SP degree.")
    g.add_argument(
        "--denoiser-ulysses", type=int, default=None, help="Denoiser Ulysses degree."
    )
    g.add_argument(
        "--denoiser-ring", type=int, default=None, help="Denoiser Ring degree."
    )
    g.add_argument("--decoder-tp", type=int, default=None, help="Decoder TP degree.")

    # P2P transfer engine
    g.add_argument(
        "--disagg-transfer-pool-size",
        type=int,
        default=256 * 1024 * 1024,
        help="P2P transfer buffer pool size in bytes (default: 256 MiB).",
    )
    g.add_argument(
        "--disagg-p2p-hostname",
        type=str,
        default="127.0.0.1",
        help="RDMA-reachable hostname/IP of this instance (default: 127.0.0.1).",
    )
    g.add_argument(
        "--disagg-ib-device",
        type=str,
        default=None,
        help="InfiniBand device for RDMA transfers (e.g., mlx5_0).",
    )
```
**EN:** This block defines function `add_disagg_cli_args`. Register all disaggregated-diffusion CLI arguments as a group. Key calls include `parser.add_argument_group`, `g.add_argument`, and `RoleType.choices`. Parameters such as `parser` drive the behavior in this section.
**CN:** 该代码块定义了函数 `add_disagg_cli_args`。 它用于处理 add disagg cli args 相关逻辑。 关键调用包括 `parser.add_argument_group`、`g.add_argument` 和 `RoleType.choices`。 本段逻辑主要由 `parser` 等参数驱动。

### Lines 190-193: `convert_disagg_role_string` implementation / `convert_disagg_role_string` 实现
```python
def convert_disagg_role_string(kwargs: dict) -> None:
    """Convert ``disagg_role`` from string to ``RoleType`` enum in-place."""
    if "disagg_role" in kwargs and isinstance(kwargs["disagg_role"], str):
        kwargs["disagg_role"] = RoleType.from_string(kwargs["disagg_role"])
```
**EN:** This block defines function `convert_disagg_role_string`. Convert ``disagg_role`` from string to ``RoleType`` enum in-place. Key calls include `isinstance`, and `RoleType.from_string`. The implementation branches on conditions. Parameters such as `kwargs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `convert_disagg_role_string`。 它用于转换disagg role string。 关键调用包括 `isinstance` 和 `RoleType.from_string`。 实现中包含条件分支。 本段逻辑主要由 `kwargs` 等参数驱动。

## Key Concepts / 关键概念
- `DisaggArgsMixin`: Methods for disaggregated diffusion, mixed into ``ServerArgs``. / 核心类，用于封装 disagg args mixin 相关行为。
- `add_disagg_cli_args`: Register all disaggregated-diffusion CLI arguments as a group. / 顶层函数，用于处理 add disagg cli args 相关逻辑。
- `convert_disagg_role_string`: Convert ``disagg_role`` from string to ``RoleType`` enum in-place. / 顶层函数，用于转换disagg role string。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `argparse`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.disaggregation.roles`

- **Total lines / 总行数**: 193

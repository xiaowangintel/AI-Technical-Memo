# launch_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/bindings/python/src/sglang_router/launch_router.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements classes `CustomHelpFormatter` and functions `launch_router`, `parse_router_args`, `main` for the Python router binding. Other gateway components import it to reuse configuration, control flow, or shared utilities. / 该模块为 Python 路由绑定层 实现了 类 `CustomHelpFormatter`与函数 `launch_router`, `parse_router_args`, `main`，其他网关组件会导入它以复用配置、控制流程或共享工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and dependencies
```python
import argparse
import logging
import sys
from typing import List, Optional

import setproctitle
from sglang_router.mini_lb import MiniLoadBalancer
from sglang_router.router_args import RouterArgs
```
**EN:** This block imports `argparse`, `logging`, `sys`, `typing`, and 3 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 10-10: Module constants and configuration
```python
logger = logging.getLogger("router")
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 12-18: Guarded setup and fallback handling
```python
try:
    from sglang_router.router import Router
except ImportError:
    Router = None
    logger.warning(
        "Rust Router is not installed, only python MiniLB (debugging only) is available"
    )
```
**EN:** This block wraps initialization in exception handling so optional functionality can fail gracefully or expose clearer diagnostics.
**CN:** 该代码块使用异常处理包裹初始化流程，使可选功能在失败时能够优雅回退或提供更清晰的诊断信息。

### Lines 21-52: function `launch_router`
```python
def launch_router(args: argparse.Namespace) -> Optional[Router]:
    """
    Launch the SGLang router with the configuration from parsed arguments.

    Args:
        args: Namespace object containing router configuration
            Can be either raw argparse.Namespace or converted RouterArgs

    Returns:
        Router instance if successful, None if failed
    """
    setproctitle.setproctitle("sglang::router")
    try:
        # Convert to RouterArgs if needed
        if not isinstance(args, RouterArgs):
            router_args = RouterArgs.from_cli_args(args)
        else:
            router_args = args

        if router_args.mini_lb:
            mini_lb = MiniLoadBalancer(router_args)
            mini_lb.start()
        else:
            if Router is None:
                raise RuntimeError("Rust Router is not installed")
            router_args._validate_router_args()
            router = Router.from_args(router_args)
            router.start()

    except Exception as e:
        logger.error(f"Error starting router: {e}")
        raise e
```
**EN:** function `launch_router` is described as: Launch the SGLang router with the configuration from parsed arguments.
**CN:** 函数 `launch_router` 封装了与 `launch router` 相关的逻辑，供本模块中的调用方复用。

### Lines 55-60: Class `CustomHelpFormatter`
```python
class CustomHelpFormatter(
    argparse.RawDescriptionHelpFormatter, argparse.ArgumentDefaultsHelpFormatter
):
    """Custom formatter that preserves both description formatting and shows defaults"""

    pass
```
**EN:** Class `CustomHelpFormatter` groups related state and behavior. It exposes 0 method(s) that implement the module's primary abstraction.
**CN:** 类 `CustomHelpFormatter` 将相关状态与行为封装在一起，提供 0 个方法来实现本模块的核心抽象。

### Lines 63-100: function `parse_router_args`
```python
def parse_router_args(args: List[str]) -> RouterArgs:
    """Parse command line arguments and return RouterArgs instance."""
    parser = argparse.ArgumentParser(
        description="""SGLang Router - High-performance request distribution across worker nodes

Usage:
This launcher enables starting a router with individual worker instances. It is useful for
multi-node setups or when you want to start workers and router separately.

Examples:
  # Regular mode
  python -m sglang_router.launch_router --worker-urls http://worker1:8000 http://worker2:8000

  # PD disaggregated mode with same policy for both
  python -m sglang_router.launch_router --pd-disaggregation \\
    --prefill http://prefill1:8000 9000 --prefill http://prefill2:8000 \\
    --decode http://decode1:8001 --decode http://decode2:8001 \\
    --policy cache_aware

  # PD mode with optional bootstrap ports
  python -m sglang_router.launch_router --pd-disaggregation \\
    --prefill http://prefill1:8000 9000 \\    # With bootstrap port
    --prefill http://prefill2:8000 none \\    # Explicitly no bootstrap port
    --prefill http://prefill3:8000 \\         # Defaults to no bootstrap port
    --decode http://decode1:8001 --decode http://decode2:8001

  # PD mode with different policies for prefill and decode
  python -m sglang_router.launch_router --pd-disaggregation \\
    --prefill http://prefill1:8000 --prefill http://prefill2:8000 \\
    --decode http://decode1:8001 --decode http://decode2:8001 \\
    --prefill-policy cache_aware --decode-policy power_of_two

    """,
        formatter_class=CustomHelpFormatter,
    )

    RouterArgs.add_cli_args(parser, use_router_prefix=False)
    return RouterArgs.from_cli_args(parser.parse_args(args), use_router_prefix=False)
```
**EN:** function `parse_router_args` is described as: Parse command line arguments and return RouterArgs instance.
**CN:** 函数 `parse_router_args` 封装了与 `parse router args` 相关的逻辑，供本模块中的调用方复用。

### Lines 103-105: function `main`
```python
def main() -> None:
    router_args = parse_router_args(sys.argv[1:])
    launch_router(router_args)
```
**EN:** function `main` encapsulates `main` logic for callers in this module.
**CN:** 函数 `main` 封装了与 `main` 相关的逻辑，供本模块中的调用方复用。

### Lines 108-109: Command-line entry point
```python
if __name__ == "__main__":
    main()
```
**EN:** This main-guard block makes the file executable as a script by invoking the module's top-level routine only when run directly.
**CN:** 这个主入口保护块让文件可以作为脚本直接执行，并仅在被直接运行时调用模块的顶层流程。

## Key Concepts / 关键概念
- Routing, worker selection, or load balancing / 路由、工作节点选择或负载均衡
- Command-line argument parsing / 命令行参数解析
- Primary classes: `CustomHelpFormatter` / 主要类：`CustomHelpFormatter`
- Reusable functions: `launch_router`, `parse_router_args`, `main` / 可复用函数：`launch_router`, `parse_router_args`, `main`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `logging`, `sys`, `typing`
- **Third-party / 第三方**: `setproctitle`
- **Internal / 内部模块**: `sglang_router.mini_lb`, `sglang_router.router`, `sglang_router.router_args`

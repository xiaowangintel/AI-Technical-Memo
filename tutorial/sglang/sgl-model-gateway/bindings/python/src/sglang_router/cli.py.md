# cli.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/bindings/python/src/sglang_router/cli.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements functions `create_parser`, `main` for the Python router binding. Other gateway components import it to reuse configuration, control flow, or shared utilities. / 该模块为 Python 路由绑定层 实现了 函数 `create_parser`, `main`，其他网关组件会导入它以复用配置、控制流程或共享工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Module header and comments
```python
#!/usr/bin/env python3
```
**EN:** These leading lines typically contain a shebang, encoding note, or comments that frame the rest of the file.
**CN:** 这些起始行通常包含 shebang、编码说明或注释，用于为后续代码提供背景。

### Lines 2-11: Module docstring
```python
"""
SGLang Model Gateway CLI

Provides convenient command-line interface for launching the router and server.

Usage:
    smg launch [args]          # Launch router only
    smg server [args]          # Launch router + server
    smg --help                 # Show help
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 13-21: Imports and dependencies
```python
import argparse
import os
import sys
from typing import List, Optional

from sglang_router.sglang_router_rs import (
    get_verbose_version_string,
    get_version_string,
)
```
**EN:** This block imports `argparse`, `os`, `sys`, `typing`, and 1 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 24-51: function `create_parser`
```python
def create_parser() -> argparse.ArgumentParser:
    """Create the main CLI parser with subcommands."""
    prog_name = os.path.basename(sys.argv[0]) if sys.argv else "smg"
    parser = argparse.ArgumentParser(
        prog=prog_name,
        description="SGLang Model Gateway - High-performance inference router",
        formatter_class=argparse.RawDescriptionHelpFormatter,
    )

    subparsers = parser.add_subparsers(dest="command", help="Available commands")

    # Launch router subcommand
    launch_parser = subparsers.add_parser(
        "launch",
        help="Launch router only (requires existing worker URLs)",
        description="Launch the SGLang router with existing worker instances",
        add_help=False,  # Let router handle --help
    )

    # Launch server + router subcommand
    server_parser = subparsers.add_parser(
        "server",
        help="Launch router and server processes together",
        description="Launch both SGLang router and server processes",
        add_help=False,  # Let server handle --help
    )

    return parser
```
**EN:** function `create_parser` is described as: Create the main CLI parser with subcommands.
**CN:** 函数 `create_parser` 封装了与 `create parser` 相关的逻辑，供本模块中的调用方复用。

### Lines 54-103: function `main`
```python
def main(argv: Optional[List[str]] = None) -> None:
    """Main CLI entry point."""
    if argv is None:
        argv = sys.argv[1:]

    # Handle version flags before parsing
    if argv and argv[0] in ["--version", "-V", "--version-verbose"]:
        if argv[0] == "--version-verbose":
            print(get_verbose_version_string())
        else:
            print(get_version_string())
        sys.exit(0)

    # Handle empty command - show help
    if not argv or argv[0] not in ["launch", "server", "-h", "--help"]:
        parser = create_parser()
        parser.print_help()
        sys.exit(1)

    parser = create_parser()
    args, unknown = parser.parse_known_args(argv)

    if args.command == "launch":
        # Import and call launch_router functions directly
        from sglang_router.launch_router import launch_router, parse_router_args

        # All router args are in unknown
        router_args = parse_router_args(unknown)
        launch_router(router_args)

    elif args.command == "server":
        # Import and call launch_server main with proper argv
        # Note: launch_server.main() uses argparse internally which reads sys.argv
        # We need to temporarily set sys.argv for compatibility
        import sglang_router.launch_server as launch_server_module

        # Preserve original sys.argv
        original_argv = sys.argv
        try:
            # All server args are in unknown
            prog_name = os.path.basename(sys.argv[0]) if sys.argv else "smg"
            sys.argv = [f"{prog_name} server"] + unknown
            launch_server_module.main()
        finally:
            # Restore original sys.argv
            sys.argv = original_argv

    else:
        parser.print_help()
        sys.exit(1)
```
**EN:** function `main` is described as: Main CLI entry point.
**CN:** 函数 `main` 封装了与 `main` 相关的逻辑，供本模块中的调用方复用。

### Lines 106-107: Command-line entry point
```python
if __name__ == "__main__":
    main()
```
**EN:** This main-guard block makes the file executable as a script by invoking the module's top-level routine only when run directly.
**CN:** 这个主入口保护块让文件可以作为脚本直接执行，并仅在被直接运行时调用模块的顶层流程。

## Key Concepts / 关键概念
- Routing, worker selection, or load balancing / 路由、工作节点选择或负载均衡
- Command-line argument parsing / 命令行参数解析
- Reusable functions: `create_parser`, `main` / 可复用函数：`create_parser`, `main`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `os`, `sys`, `typing`
- **Third-party / 第三方**: None explicitly imported / 未显式导入
- **Internal / 内部模块**: `sglang_router.launch_router`, `sglang_router.launch_server`, `sglang_router.sglang_router_rs`

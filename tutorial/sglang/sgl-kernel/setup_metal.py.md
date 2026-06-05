# setup_metal.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/setup_metal.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This build/setup script prepares the `Metal build` path, checks prerequisites, and compiles or installs native pieces when required. / 该构建/安装脚本用于准备 `Metal build` 路径，检查前置条件，并在需要时编译或安装原生组件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Comments and local context
````python
# Copyright 2026 SGLang Team. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 16-23: Imports and module setup
````python
import importlib
import os
import platform
import shutil
import subprocess
import sys
import sysconfig
from pathlib import Path
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 25-25: Constants and configuration
````python
root = Path(__file__).parent.resolve()
````
**EN:** This block defines shared constants or configuration values such as `root`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `root`），供后续函数或控制流程复用。

### Lines 28-32: Constants and configuration
````python
_BUILD_REQUIRES = [
    ("setuptools", "setuptools"),
    ("mlx", "mlx"),
    ("nanobind", "nanobind"),
]
````
**EN:** This block defines shared constants or configuration values such as `_BUILD_REQUIRES`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_BUILD_REQUIRES`），供后续函数或控制流程复用。

### Lines 35-53: `_ensure_toolchain` definition
````python
def _ensure_toolchain():
    if sys.platform != "darwin" or platform.machine() != "arm64":
        raise SystemExit("setup_metal.py only supports macOS (Apple Silicon).")
    if shutil.which("c++") is None or shutil.which("xcrun") is None:
        raise SystemExit(
            "Apple toolchain not found. Install the Xcode Command Line Tools "
            "with `xcode-select --install` (or a full Xcode install) and retry."
        )
    try:
        subprocess.check_output(
            ["xcrun", "-sdk", "macosx", "metal", "--version"],
            stderr=subprocess.STDOUT,
        )
    except (subprocess.CalledProcessError, FileNotFoundError) as exc:
        raise SystemExit(
            "Apple Metal shader compiler not found. Install a full Xcode "
            "(not just Command Line Tools) so that `xcrun -sdk macosx metal` "
            "is available, then retry."
        ) from exc
````
**EN:** This section defines `_ensure_toolchain` and implements the core logic associated with ensure toolchain.
**CN:** 该部分定义 `_ensure_toolchain`，并实现与 ensure toolchain 相关的核心逻辑。

### Lines 56-71: `_ensure_build_requires` definition
````python
def _ensure_build_requires():
    missing = []
    for import_name, pip_name in _BUILD_REQUIRES:
        try:
            importlib.import_module(import_name)
        except ImportError:
            missing.append(pip_name)
    if not missing:
        return
    print(
        f"[sgl-kernel:metal] installing build requirements: {missing}",
        flush=True,
    )
    subprocess.check_call(
        [sys.executable, "-m", "pip", "install", "--upgrade", *missing]
    )
````
**EN:** This section defines `_ensure_build_requires` and implements the core logic associated with ensure build requires.
**CN:** 该部分定义 `_ensure_build_requires`，并实现与 ensure build requires 相关的核心逻辑。

### Lines 72-74: Comments and local context
````python


# Section 1: Prerequisites
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 75-75: Top-level logic
````python
_ensure_toolchain()
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 76-76: Top-level logic
````python
_ensure_build_requires()
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 77-77: Top-level logic
````python
os.chdir(root)
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 78-80: Comments and local context
````python


# Section 2: Build and install
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 81-82: Imports and module setup
````python
from setuptools import Extension, find_packages, setup  # noqa: E402
from setuptools.command.build_ext import build_ext  # noqa: E402
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 85-89: `_get_version` definition
````python
def _get_version():
    with open(root / "pyproject.toml") as f:
        for line in f:
            if line.startswith("version"):
                return line.split("=")[1].strip().strip('"')
````
**EN:** This section defines `_get_version` and implements the core logic associated with get version.
**CN:** 该部分定义 `_get_version`，并实现与 get version 相关的核心逻辑。

### Lines 92-93: Constants and configuration
````python
operator_namespace = "sgl_kernel"
metallib_name = "sgl_metal_kernels.metallib"
````
**EN:** This block defines shared constants or configuration values such as `operator_namespace`, `metallib_name`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `operator_namespace`, `metallib_name`），供后续函数或控制流程复用。

### Lines 94-96: Comments and local context
````python

# Metal shader sources (compiled with `xcrun metal`) and C++ host sources
# (compiled with `c++`). Add new kernels by appending to these lists.
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 97-102: Constants and configuration
````python
metal_shader_sources = [
    "csrc/metal/placeholder.metal",
]
cxx_sources = [
    "csrc/metal/placeholder.cpp",
]
````
**EN:** This block defines shared constants or configuration values such as `metal_shader_sources`, `cxx_sources`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `metal_shader_sources`, `cxx_sources`），供后续函数或控制流程复用。

### Lines 103-105: Comments and local context
````python

# Header search paths shared by both the Metal shader compiler and the C++
# host compiler.
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 106-114: Constants and configuration
````python
include_dirs = [
    root / "csrc",
    root / "csrc" / "metal",
]

cxx_flags = ["-std=c++17", "-O3", "-fvisibility=hidden"]
metal_flags = ["-O3"]
frameworks = ["Metal", "Foundation", "QuartzCore"]
libraries = ["mlx"]
````
**EN:** This block defines shared constants or configuration values such as `include_dirs`, `cxx_flags`, `metal_flags`, `frameworks`, `libraries`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `include_dirs`, `cxx_flags`, `metal_flags`, `frameworks`, `libraries`），供后续函数或控制流程复用。

### Lines 117-277: `BuildMetalExtension` definition
````python
class BuildMetalExtension(build_ext):
    def build_extension(self, ext):
        if sys.platform != "darwin" or platform.machine() != "arm64":
            raise RuntimeError("setup_metal.py only supports macOS (Apple Silicon).")

        ext_path = Path(self.get_ext_fullpath(ext.name))
        ext_path.parent.mkdir(parents=True, exist_ok=True)

        # Use ccache for the C++ compiles when it is on PATH. ccache does not
        # support `.metal` sources (unsupported source language), so the Metal
        # shader compile is left untouched.
        ccache = shutil.which("ccache")
        cxx_cmd = [ccache, "c++"] if ccache else ["c++"]
        if ccache:
            print(f"[sgl-kernel:metal] using ccache at {ccache}", flush=True)

        python_exe = Path(sys.executable)
        python_include = Path(sysconfig.get_paths()["include"])
        python_lib = Path(sysconfig.get_config_var("LIBDIR"))
        # Match the deployment target that Python itself was built against
        # unless the user overrides it. MLX's prebuilt wheels may require a
        # higher minimum; in that case set MACOSX_DEPLOYMENT_TARGET explicitly.
        deployment_target = os.environ.get(
            "MACOSX_DEPLOYMENT_TARGET",
            str(sysconfig.get_config_var("MACOSX_DEPLOYMENT_TARGET") or "11.0"),
        )

        def _python_eval(expr: str) -> str:
            return subprocess.check_output(
                [str(python_exe), "-c", expr], text=True
            ).strip()

        nanobind_dir = Path(
            _python_eval("import nanobind; print(nanobind.__path__[0])")
        )
        mlx_dir = Path(_python_eval("import mlx.core as mx; print(mx.__file__)"))
        mlx_site = mlx_dir.parent
        mlx_include = mlx_site / "include"
        mlx_lib = mlx_site / "lib"

        generated_dir = root / "build" / "metal"
        generated_dir.mkdir(parents=True, exist_ok=True)

        metallib_path = generated_dir / metallib_name
        metal_std = os.environ.get("SGL_METAL_STD", "metal3.1")

        ext_include_dirs = [Path(p) for p in (ext.include_dirs or [])]
        host_includes = [
            python_include,
            nanobind_dir / "include",
            nanobind_dir / "ext" / "robin_map" / "include",
            mlx_include,
            mlx_include / "metal_cpp",
        ]
        all_includes = ext_include_dirs + host_includes
        include_args = [f"-I{p}" for p in all_includes]
        # `xcrun metal` accepts `-I` for header search; reuse the project
        # include dirs so shaders can include shared MSL headers.
        metal_include_args = [f"-I{p}" for p in ext_include_dirs]

        if not metal_shader_sources:
            raise RuntimeError("metal_shader_sources is empty; nothing to compile")

        air_paths = []
        for rel in metal_shader_sources:
            metal_src = root / rel
            if not metal_src.is_file():
                raise RuntimeError(f"metal shader source not found: {metal_src}")
            air_path = generated_dir / (metal_src.stem + ".air")
            self.spawn(
                [
                    "xcrun",
                    "-sdk",
                    "macosx",
                    "metal",
                    f"-std={metal_std}",
                    *metal_flags,
                    *metal_include_args,
                    "-c",
                    str(metal_src),
                    "-o",
                    str(air_path),
                ]
            )
            air_paths.append(str(air_path))

        self.spawn(
            [
                "xcrun",
                "-sdk",
                "macosx",
                "metallib",
                *air_paths,
                "-o",
                str(metallib_path),
            ]
        )

        cflags = [
            *cxx_flags,
            f"-mmacosx-version-min={deployment_target}",
            *include_args,
        ]

        ldflags = [
            "-shared",
            "-undefined",
            "dynamic_lookup",
            f"-mmacosx-version-min={deployment_target}",
            f"-L{python_lib}",
            f"-L{mlx_lib}",
            f"-Wl,-rpath,{mlx_lib}",
            *[f"-l{lib}" for lib in libraries],
            *[arg for fw in frameworks for arg in ("-framework", fw)],
        ]

        objects = []
        for src in ext.sources:
            src_path = Path(src)
            obj_path = generated_dir / (src_path.stem + ".o")
            compile_cmd = [
                *cxx_cmd,
                *cflags,
                "-c",
                str(src_path),
                "-o",
                str(obj_path),
            ]
            self.spawn(compile_cmd)
            objects.append(str(obj_path))

        nanobind_src = nanobind_dir / "src" / "nb_combined.cpp"
        nanobind_obj = generated_dir / "nb_combined.o"
        nanobind_cmd = [
            *cxx_cmd,
            *cflags,
            "-DNB_COMPACT_ASSERTIONS",
            "-DNB_BUILD",
            "-DNB_SHARED",
            "-c",
            str(nanobind_src),
            "-o",
            str(nanobind_obj),
        ]
        self.spawn(nanobind_cmd)
        objects.append(str(nanobind_obj))

        link_cmd = [
            "c++",
            *objects,
            *ldflags,
            "-o",
            str(ext_path),
        ]
        self.spawn(link_cmd)

        # Stage the metallib next to the freshly-linked extension so that
        # `install_lib` picks it up via `package_data={"sgl_kernel": ["*.metallib"]}`.
        staged_metallib = ext_path.parent / metallib_path.name
        if metallib_path.resolve() != staged_metallib.resolve():
            shutil.copy2(metallib_path, staged_metallib)
````
**EN:** This section defines the class `BuildMetalExtension`, grouping related state and behavior around Build Metal Extension.
**CN:** 该部分定义类 `BuildMetalExtension`，把与 Build Metal Extension 相关的状态和行为组织在一起。

### Lines 280-287: Constants and configuration
````python
ext_modules = [
    Extension(
        name=f"{operator_namespace}._metal",
        sources=cxx_sources,
        include_dirs=[str(p) for p in include_dirs],
        language="c++",
    )
]
````
**EN:** This block defines shared constants or configuration values such as `ext_modules`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `ext_modules`），供后续函数或控制流程复用。

### Lines 289-298: Top-level logic
````python
setup(
    name="sglang-kernel",
    version=_get_version(),
    packages=find_packages(where="python"),
    package_dir={"": "python"},
    package_data={"sgl_kernel": ["*.metallib"]},
    include_package_data=True,
    ext_modules=ext_modules,
    cmdclass={"build_ext": BuildMetalExtension},
)
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

## Key Concepts / 关键概念
- **Role / 角色**: Build helper / 构建辅助脚本
- **Primary symbols / 主要符号**: `_ensure_toolchain`, `_ensure_build_requires`, `_get_version`, `BuildMetalExtension`
- **Core technologies / 核心技术**: toolchain subprocess control / 工具链子进程控制

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `importlib`, `os`, `pathlib`, `platform`, `setuptools`, `setuptools.command.build_ext`, `shutil`, `subprocess`, `sys`, `sysconfig`

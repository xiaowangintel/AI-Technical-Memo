# incremental_build.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/incremental_build.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Presents a faster workflow for iterating on `csrc/` kernels by keeping Python installed in editable mode while rebuilding only affected native components through CMake.
- **CN:** 本文给出了更快的 `csrc/` 内核迭代流程：Python 侧保持 editable 安装，只通过 CMake 重建受影响的原生组件。
- **EN:** The guide covers prerequisites, auto-generated `CMakeUserPresets.json`, configure/build/install commands, artifact verification, and tuning advice for rebuild speed and stability.
- **CN:** 文档覆盖了前置条件、自动生成的 `CMakeUserPresets.json`、配置/编译/安装命令、产物校验，以及关于重建速度和稳定性的调优建议。

## Key Concepts / 关键概念
- **EN:** Editable install — The workflow assumes Python code is already installed from source in editable mode.
- **CN:** Editable 安装 — 该流程假设 Python 代码已通过源码 editable 方式安装。
- **EN:** `CMakeUserPresets.json` — User presets capture compiler paths, cache launchers, install prefix, and build parallelism.
- **CN:** `CMakeUserPresets.json` — 用户预设文件记录编译器路径、缓存工具、安装前缀以及并行构建参数。
- **EN:** Install target — Using `cmake --build --target install` copies rebuilt shared libraries back into the source tree.
- **CN:** 安装目标 — 使用 `cmake --build --target install` 会把重建后的共享库复制回源码树。
- **EN:** Rebuild tuning — ccache, job counts, and occasional clean builds are key to a stable fast loop.
- **CN:** 重建调优 — ccache、并行度设置以及必要时的全量清理，是稳定快速迭代的关键。

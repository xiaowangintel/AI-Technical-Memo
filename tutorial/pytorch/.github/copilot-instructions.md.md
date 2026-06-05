# copilot-instructions.md — Documentation Analysis / 文档分析

## Source / 来源
- **File / 文件**: `.github/copilot-instructions.md`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Stores repository-level GitHub metadata, automation policies, and contribution-facing configuration.
- **用途 (CN)**: 存放仓库级 GitHub 元数据、自动化策略以及面向贡献者的配置。

## Content Analysis / 内容分析
### Lines 1-13 / 第 1-13 行

````markdown
# PyTorch Copilot Instructions

This is the PyTorch machine learning framework codebase. These instructions help AI agents navigate and contribute effectively.

## Architecture Overview

### Core Components

- **c10/** - Core library (C++-10 compatible) for essential, binary-size-conscious functionality
- **aten/** - ATen tensor library (C++), PyTorch's foundation without autograd
  - `aten/src/ATen/native/` - Modern operator implementations (CPU/CUDA/MPS/sparse)
  - `aten/src/ATen/native/native_functions.yaml` - **Critical**: Declarative operator registry
- **torch/** - Python bindings and public API
````

- EN: This section introduces headings that organize the narrative or instructions; lists actionable items, options, or policy bullets.
- CN: 该部分通过标题组织叙述或操作说明；列出可执行事项、选项或策略要点。

### Lines 14-25 / 第 14-25 行

````markdown
  - `torch/csrc/` - C++ Python bindings (hand-written and generated)
  - `torch/csrc/autograd/` - Reverse-mode automatic differentiation
  - `torch/csrc/jit/` - TorchScript JIT compiler
- **torchgen/** - Code generation tooling that reads `native_functions.yaml`
- **tools/** - Build scripts, autograd derivatives, code generation

### The Code Generation Workflow

**Most operator changes require editing `native_functions.yaml`**, not direct C++ files. This YAML file:
1. Declares operator signatures, variants (function/method), and dispatch behavior
2. Gets processed by `torchgen/` to generate C++/Python bindings
3. Produces headers in `build/aten/src/ATen/` during compilation
````

- EN: This section introduces headings that organize the narrative or instructions; lists actionable items, options, or policy bullets.
- CN: 该部分通过标题组织叙述或操作说明；列出可执行事项、选项或策略要点。

### Lines 27-38 / 第 27-38 行

````markdown
Example entry structure:
```yaml
- func: my_op(Tensor self, Scalar alpha=1) -> Tensor
  variants: function, method
  dispatch:
    CPU: my_op_cpu
    CUDA: my_op_cuda
```

After editing `native_functions.yaml`, implement kernels in `aten/src/ATen/native/` (see `aten/src/ATen/native/README.md`).

## Development Workflows
````

- EN: This section introduces headings that organize the narrative or instructions; lists actionable items, options, or policy bullets.
- CN: 该部分通过标题组织叙述或操作说明；列出可执行事项、选项或策略要点。

### Lines 40-52 / 第 40-52 行

````markdown
### Building from Source

**Never run `setup.py` directly** - use pip with editable install:
```bash
python -m pip install --no-build-isolation -v -e .
```

Speed up builds:
- `DEBUG=1` - Debug symbols with `-g -O0`
- `USE_CUDA=0` - Skip CUDA compilation
- `BUILD_TEST=0` - Skip C++ test binaries
- Install `ninja` (`pip install ninja`) for faster builds
- Use `ccache` for incremental compilation caching
````

- EN: This section introduces headings that organize the narrative or instructions; lists actionable items, options, or policy bullets.
- CN: 该部分通过标题组织叙述或操作说明；列出可执行事项、选项或策略要点。

### Lines 54-65 / 第 54-65 行

````markdown
Rebuild specific targets: `(cd build && ninja <target>)`

### Testing

**Critical**: DO NOT run entire test suites. Run specific tests only:
```bash
python test/test_torch.py TestTorch.test_specific_case
```

**Test structure**: All tests use `torch.testing._internal.common_utils`:
```python
from torch.testing._internal.common_utils import run_tests, TestCase
````

- EN: This section introduces headings that organize the narrative or instructions.
- CN: 该部分通过标题组织叙述或操作说明。

### Lines 67-78 / 第 67-78 行

````markdown
class TestFeature(TestCase):
    def test_something(self):
        # Use self.assertEqual for tensor comparisons
        pass

if __name__ == "__main__":
    run_tests()
```

**For bug fixes**: Create a standalone reproduction script first, verify it fails, then fix and add to appropriate test file.

### Linting
````

- EN: This section introduces headings that organize the narrative or instructions.
- CN: 该部分通过标题组织叙述或操作说明。

### Lines 80-91 / 第 80-91 行

````markdown
Run linter (not pre-commit): `lintrunner -a` (auto-applies fixes)

## Project-Specific Conventions

### Memory and Storage
- **Storage is never nullptr** (but `StorageImpl.data` may be nullptr for unallocated outputs)
- CUDA device info lives in storage objects

### Python-C++ Integration (`torch/csrc/`)
- Always include `Python.h` **first** to avoid `_XOPEN_SOURCE` redefinition errors
- Use `pybind11::gil_scoped_acquire` before calling Python API or using `THPObjectPtr`
- Wrap entry points with `HANDLE_TH_ERRORS` / `END_HANDLE_TH_ERRORS` for exception conversion
````

- EN: This section introduces headings that organize the narrative or instructions; lists actionable items, options, or policy bullets.
- CN: 该部分通过标题组织叙述或操作说明；列出可执行事项、选项或策略要点。

### Lines 93-104 / 第 93-104 行

````markdown
### Dispatch System
- PyTorch uses operator dispatch to route calls to backend-specific kernels
- Prefer `CompositeExplicitAutograd` dispatch when writing device-agnostic compound ops
- See `aten/src/ATen/native/README.md` for dispatch keyword guidance

## Git Workflow (AI Agent Specific)

When preparing PRs from this environment:
```bash
git stash -u
git reset --hard $(cat /tmp/orig_work.txt)  # Reset to LOCAL branch
git stash pop
````

- EN: This section introduces headings that organize the narrative or instructions; lists actionable items, options, or policy bullets.
- CN: 该部分通过标题组织叙述或操作说明；列出可执行事项、选项或策略要点。

### Lines 105-117 / 第 105-117 行

````markdown
# Resolve conflicts if necessary
```

## Common Gotchas

1. **Editing generated files** - If it's in `build/`, don't edit it. Edit the source template or `native_functions.yaml`
2. **NVCC template compilation** - NVCC is stricter about C++ than gcc/clang; code working on Linux may fail Windows CI
3. **Windows symbol visibility** - Use `TORCH_API` macros for exported symbols (required on Windows, optional on Linux)
4. **No internet access** - DO NOT attempt to install dependencies during development

## Key Files Reference

- `AGENTS.md` - Instructions specific to AI coding agents
````

- EN: This section introduces headings that organize the narrative or instructions; lists actionable items, options, or policy bullets.
- CN: 该部分通过标题组织叙述或操作说明；列出可执行事项、选项或策略要点。

### Lines 118-123 / 第 118-123 行

````markdown
- `CONTRIBUTING.md` - Comprehensive human contributor guide
- `GLOSSARY.md` - Terminology (ATen, kernels, operations, JIT, TorchScript)
- `aten/src/ATen/native/README.md` - Operator implementation guide
- `tools/autograd/derivatives.yaml` - Gradient definitions for autograd

## Performance Debugging
````

- EN: This section introduces headings that organize the narrative or instructions; lists actionable items, options, or policy bullets.
- CN: 该部分通过标题组织叙述或操作说明；列出可执行事项、选项或策略要点。

### Lines 125-125 / 第 125-125 行

````markdown
Use `TORCH_SHOW_CPP_STACKTRACES=1` for C++ traces in Python errors. For profiling, prefer `py-spy` over manual instrumentation.
````

- EN: This section explains repository context in prose form.
- CN: 该部分以文字形式解释仓库上下文。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Configures native build inputs, optional backends, and dependency discovery at configure time.
  CN: 在配置阶段设置原生构建输入、可选后端以及依赖发现逻辑。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Captures repository governance and contributor-facing policies in declarative configuration.
  CN: 以声明式配置的形式记录仓库治理规则和面向贡献者的策略。

## Dependencies / 依赖关系
- Headings or named sections / 标题或命名章节: `PyTorch Copilot Instructions`, `Architecture Overview`, `Core Components`, `The Code Generation Workflow`, `Development Workflows`, `Building from Source`, `Testing`, `Linting`, `Project-Specific Conventions`, `Memory and Storage`, ...
- Referenced paths / 引用路径: `aten/src/ATen/native/`, `CPU/CUDA/MPS/sparse`, `aten/src/ATen/native/native_functions.yaml`, `torch/csrc/`, `torch/csrc/autograd/`, `torch/csrc/jit/`, `function/method`, `/Python`, `build/aten/src/ATen/`, `aten/src/ATen/native/README.md`, ...
- URLs / 链接: none

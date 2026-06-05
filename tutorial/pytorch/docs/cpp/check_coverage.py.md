# check_coverage.py — Code Analysis / 代码分析
## Source / 来源
- File: `docs/cpp/check_coverage.py`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Check documentation coverage and report missing C++ API references or bindings.
- 用途 (CN): 检查文档覆盖率，并报告缺失的 C++ API 引用或绑定。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1 / 第 1-1 行
```python
#!/usr/bin/env python3
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。

### Lines 2-17 / 第 2-17 行
```python
"""C++ API documentation coverage checker.

Auto-discovers public C++ APIs from Doxygen XML output and checks which ones
are documented in the RST source files via Breathe or Sphinx C++ domain
directives.

Uses an exclusion list (EXCLUDED_APIS) to skip internal/detail symbols that
don't need public documentation, rather than maintaining a hardcoded allowlist.

Additionally checks built HTML for broken formatting (empty pages,
unresolved directives, rendering errors).

Usage:
    python check_coverage.py                  # RST coverage + HTML checks
    python check_coverage.py --coverxygen     # also run coverxygen on Doxygen XML
"""
```
- EN: This segment is module-level descriptive text. It frames the file around: C++ API documentation coverage checker. It configures or cooperates with Sphinx documentation machinery.
- CN: 这一段是模块级说明文字，用于界定文件主题：C++ API documentation coverage checker. 它会配置或配合 Sphinx 文档机制。

### Lines 19-19 / 第 19-19 行
```python
import argparse
```
- EN: This segment imports `argparse`, establishing dependencies needed by the rest of the script. It participates in command-line argument parsing.
- CN: 这一段导入了 `argparse`，为脚本后续逻辑建立依赖。 它参与命令行参数解析。

### Lines 20-20 / 第 20-20 行
```python
import re
```
- EN: This segment imports `re`, establishing dependencies needed by the rest of the script.
- CN: 这一段导入了 `re`，为脚本后续逻辑建立依赖。

### Lines 21-21 / 第 21-21 行
```python
import subprocess
```
- EN: This segment imports `subprocess`, establishing dependencies needed by the rest of the script. It launches external commands or tools.
- CN: 这一段导入了 `subprocess`，为脚本后续逻辑建立依赖。 它会调用外部命令或工具。

### Lines 22-22 / 第 22-22 行
```python
import sys
```
- EN: This segment imports `sys`, establishing dependencies needed by the rest of the script.
- CN: 这一段导入了 `sys`，为脚本后续逻辑建立依赖。

### Lines 23-23 / 第 23-23 行
```python
import xml.etree.ElementTree as ET
```
- EN: This segment imports `xml.etree.ElementTree`, establishing dependencies needed by the rest of the script.
- CN: 这一段导入了 `xml.etree.ElementTree`，为脚本后续逻辑建立依赖。

### Lines 24-24 / 第 24-24 行
```python
from pathlib import Path
```
- EN: This segment imports `pathlib.Path`, establishing dependencies needed by the rest of the script. It manipulates filesystem paths and generated files.
- CN: 这一段导入了 `pathlib.Path`，为脚本后续逻辑建立依赖。 它会处理文件系统路径以及生成出的文件。

### Lines 25-28 / 第 25-28 行
```python


# ─── Paths ───────────────────────────────────────────────────────────────────
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。

### Lines 29-29 / 第 29-29 行
```python
SCRIPT_DIR = Path(__file__).resolve().parent
```
- EN: This segment defines or updates `SCRIPT_DIR`, which shapes later behavior in the file. It manipulates filesystem paths and generated files.
- CN: 这一段定义或更新了 `SCRIPT_DIR`，从而影响文件后续行为。 它会处理文件系统路径以及生成出的文件。

### Lines 30-30 / 第 30-30 行
```python
SOURCE_DIR = SCRIPT_DIR / "source"
```
- EN: This segment defines or updates `SOURCE_DIR`, which shapes later behavior in the file.
- CN: 这一段定义或更新了 `SOURCE_DIR`，从而影响文件后续行为。

### Lines 31-31 / 第 31-31 行
```python
BUILD_HTML = SCRIPT_DIR / "build" / "html"
```
- EN: This segment defines or updates `BUILD_HTML`, which shapes later behavior in the file.
- CN: 这一段定义或更新了 `BUILD_HTML`，从而影响文件后续行为。

### Lines 32-32 / 第 32-32 行
```python
BUILD_XML = SCRIPT_DIR / "build" / "xml"
```
- EN: This segment defines or updates `BUILD_XML`, which shapes later behavior in the file.
- CN: 这一段定义或更新了 `BUILD_XML`，从而影响文件后续行为。

### Lines 33-33 / 第 33-33 行
```python
COVERAGE_OUTPUT = SCRIPT_DIR / "cpp_coverage.txt"
```
- EN: This segment defines or updates `COVERAGE_OUTPUT`, which shapes later behavior in the file.
- CN: 这一段定义或更新了 `COVERAGE_OUTPUT`，从而影响文件后续行为。

### Lines 34-34 / 第 34-34 行
```python
HTML_REPORT = SCRIPT_DIR / "cpp_html_issues.txt"
```
- EN: This segment defines or updates `HTML_REPORT`, which shapes later behavior in the file.
- CN: 这一段定义或更新了 `HTML_REPORT`，从而影响文件后续行为。

### Lines 35-39 / 第 35-39 行
```python

# ─── Inclusion override ──────────────────────────────────────────────────────
# Symbols that match an exclusion pattern but should still be tracked.
# Use this for "internal" APIs that are widely used as public API.
# Add the fully-qualified symbol name here and it will bypass all exclusions.
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。

### Lines 40-42 / 第 40-42 行
```python
INCLUDED_SYMBOLS: set[str] = {
    # Example: "c10::IValue" would track it even though c10::IValue is excluded
}
```
- EN: This segment defines or updates `INCLUDED_SYMBOLS`, which shapes later behavior in the file.
- CN: 这一段定义或更新了 `INCLUDED_SYMBOLS`，从而影响文件后续行为。

### Lines 43-48 / 第 43-48 行
```python

# ─── Exclusion list ──────────────────────────────────────────────────────────
# Symbols that should NOT be flagged as missing documentation.
# Add internal, detail, or otherwise non-public symbols here.
# Note: INCLUDED_SYMBOLS takes priority over these exclusions.
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node. It uses pattern matching to transform or inspect text.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。 它使用模式匹配来转换或检查文本。

### Lines 49-128 / 第 49-128 行
```python
EXCLUDED_PATTERNS = [
    # Internal/detail namespaces
    r".*::detail::.*",
    r".*::detail_::.*",
    r"torch::python::.*",
    # Underscore-prefixed internal classes
    r".*::_\w+",
    # Enum helper structs
    r"torch::enumtype::.*",
    # OptimizerCloneableOptions SFINAE helpers
    r"torch::optim::OptimizerCloneableOptions::.*",
    # Internal optimizer state/options cloneable helpers
    r"torch::optim::OptimizerCloneable.*",
    # Error classes (c10 exceptions)
    r"c10::.*Error$",
    r"c10::ErrorAlwaysShowCppStacktrace",
    # Warning internals
    r"c10::Warning.*",
    r"c10::WarningHandler",
    r"c10::WarningUtils::.*",
    # c10 IValue internals
    r"c10::IValue::.*",
    r"c10::IValue",
    r"c10::WeakIValue",
    r"c10::ivalue::.*",
    r"c10::StrongTypePtr",
    r"c10::WeakTypePtr",
    r"c10::WeakOrStrongTypePtr",
    r"c10::WeakOrStrongCompilationUnit",
    r"c10::Capsule",
    r"c10::OptionalArray",
    r"c10::StreamData3",
    # OrderedDict::Item (internal helper)
    r"torch::OrderedDict::Item",
    # ExpandingArray (internal template utility)
    r"torch::ExpandingArray.*",
    # IMethod (internal)
    r"torch::IMethod",
    # CustomClassHolder (internal base)
    r"torch::CustomClassHolder",
    # NodeGuard (internal autograd)
    r"torch::autograd::NodeGuard",
    # Autograd internals
    r"torch::autograd::CppNode",
    r"torch::autograd::ExtractVariables",
    r"torch::autograd::Node",
    r"torch::autograd::Node::.*",
    r"torch::autograd::TraceableFunction",
    r"torch::autograd::TypeAndSize",
    # Sequencer internals
    r"torch::data::.*::detail::.*",
    # cuDNN descriptor internals
    r"at::native::ActivationDescriptor",
    r"at::native::ConvolutionDescriptor",
    r"at::native::SpatialTransformerDescriptor",
    r"at::native::DropoutDescriptor",
    r"at::native::RNNDataDescriptor",
    r"at::native::DftiDescriptor",
    r"at::native::DescriptorDeleter",
    r"at::native::DftiDescriptorDeleter",
    r"at::native::RNNDescriptor",
    # ATen internals
    r"at::OptionalTensorRef",
    r"at::TensorRef",
    # at::cuda internals (allocator, workspace, cublas)
    r"at::cuda::WorkspaceMapWithMutex",
    r"at::cuda::clearCublasWorkspaces.*",
    r"at::cuda::cublas_handle_stream_to_workspace",
    r"at::cuda::cublaslt_handle_stream_to_workspace",
    r"at::cuda::getCUDABlasLt.*",
    r"at::cuda::getCUDADeviceAllocator",
    r"at::cuda::getChosenWorkspaceSize",
    r"at::cuda::getNumGPUs",
    r"at::cuda::is_available",
    r"at::cuda::warp_size",
    # jit namespace (deprecated)
    r"torch::jit::.*",
    # Operators that are just operator<< or operator>>
    r".*::operator<<",
    r".*::operator>>",
```
- EN: This segment defines or updates `EXCLUDED_PATTERNS`, which shapes later behavior in the file. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段定义或更新了 `EXCLUDED_PATTERNS`，从而影响文件后续行为。 它会与 PyTorch 模块、符号或示例交互。

### Lines 129-208 / 第 129-208 行
```python
    r".*::operator==",
    r".*::operator!=",
    # Internal serialize helpers
    r"torch::optim::serialize",
    r"torch::optim::detail::.*",
    # Reduction enum helpers
    r"torch::nn::reduction",
    r"torch::nn::log_target",
    # Internal module utils
    r"torch::nn::modules::utils::.*",
    # Internal c10 helpers
    r"c10::detail::.*",
    r"c10::detail_::.*",
    r"c10::makeArrayRef",
    r"c10::checkObjectSortSchema",
    r"c10::getGreaterThanComparator",
    r"c10::getLessThanComparator",
    r"c10::value_or_else",
    r"c10::warn",
    r"c10::GetExceptionString",
    # torch::detail
    r"torch::detail::.*",
    # Internal data shuttle/queue
    r"torch::data::detail::.*",
    # DataLoaderBase internal types
    r"torch::data::DataLoaderBase::.*",
    r"torch::data::WorkerException",
    r"torch::data::FullDataLoaderOptions",
    # Template specializations of Stack
    r"torch::data::transforms::Stack< .*>",
    # Example partial specialization
    r"torch::data::Example< .*>",
    # Doxygen internal macros
    r"DEFINE_CASE",
    r"DEFINE_TAG",
    r"COUNT_TAG",
    r"TRUTH_TABLE_ENTRY",
    r"C10_EXPAND_MSVC_WORKAROUND",
    r"TORCH_FORALL_TAGS",
    # Non-public torch::nn functions (module stream operators, etc.)
    r"torch::nn::operator.*",
    # AnyModule/AnyValue internal holders
    r"torch::nn::AnyModuleHolder.*",
    r"torch::nn::AnyModulePlaceholder",
    r"torch::nn::AnyValue.*",
    r"torch::nn::NamedAnyModule",
    # Internal base classes (users use the derived classes)
    r"torch::nn::ConvNdImpl",
    r"torch::nn::ConvTransposeNdImpl",
    r"torch::nn::BatchNormImplBase",
    r"torch::nn::NormImplBase",
    r"torch::nn::InstanceNormImpl",
    r"torch::nn::MaxPoolImpl",
    r"torch::nn::AvgPoolImpl",
    r"torch::nn::AdaptiveAvgPoolImpl",
    r"torch::nn::AdaptiveMaxPoolImpl",
    r"torch::nn::MaxUnpoolImpl",
    r"torch::nn::LPPoolImpl",
    r"torch::nn::ConstantPadImpl",
    r"torch::nn::ReflectionPadImpl",
    r"torch::nn::ReplicationPadImpl",
    r"torch::nn::ZeroPadImpl",
    r"torch::nn::FractionalMaxPoolImpl",
    # nn::functions internal namespace
    r"torch::nn::functions::.*",
    # AdaptiveLogSoftmaxWithLoss (niche, rarely used in C++)
    r"torch::nn::AdaptiveLogSoftmaxWithLoss.*",
    r"torch::nn::ASMoutput",
    # CrossMapLRN2d (niche)
    r"torch::nn::CrossMapLRN2d.*",
    # _out function variants (documented alongside the main function)
    r"torch::special::.*_out",
    r"torch::fft::.*_out",
    # torch internal helpers
    r"torch::InitLambda",
    r"torch::dispatch",
    r"torch::equal_if_defined",
    r"torch::getAllCustomClassesNames",
    r"torch::init",
    r"torch::make_custom_class",
```
- EN: This segment defines or updates `EXCLUDED_PATTERNS`, which shapes later behavior in the file. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段定义或更新了 `EXCLUDED_PATTERNS`，从而影响文件后续行为。 它会与 PyTorch 模块、符号或示例交互。

### Lines 209-223 / 第 209-223 行
```python
    r"torch::selective_class_",
    r"torch::pickle_load",
    r"torch::pickle_save",
    r"torch::schema",
    r"torch::nativert::.*",
    # RNNCellOptionsBase (internal base)
    r".*::RNNCellOptionsBase",
    # Unnamespaced Options structs (indexed without namespace by Doxygen)
    r"^[A-Z]\w+Options$",
    # Unnamespaced classes without namespace (Doxygen quirk)
    r"^TransformerDecoderLayer$",
    r"^TransformerDecoderLayerOptions$",
    # functional namespace internal options structs
    r"functional::.*FuncOptions",
]
```
- EN: This segment defines or updates `EXCLUDED_PATTERNS`, which shapes later behavior in the file. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段定义或更新了 `EXCLUDED_PATTERNS`，从而影响文件后续行为。 它会与 PyTorch 模块、符号或示例交互。

### Lines 224-225 / 第 224-225 行
```python

# Specific symbols to exclude (exact match)
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。

### Lines 226-277 / 第 226-277 行
```python
EXCLUDED_SYMBOLS = {
    # Internal / not useful to document individually
    "torch::data::datasets::map",
    "torch::data::datasets::make_shared_dataset",
    "torch::data::datasets::operator<<",
    "torch::data::datasets::operator>>",
    "torch::enumtype::get_enum_name",
    "torch::enumtype::reduction_get_enum",
    "torch::autograd::_wrap_outputs",
    "torch::autograd::check_variable_result",
    "torch::autograd::CppNode_apply_functional",
    "torch::autograd::CppNode_apply_functional_ivalue",
    "torch::autograd::forward_ad::enter_dual_level",
    "torch::autograd::forward_ad::exit_dual_level",
    "torch::autograd::any_variable_requires_grad",
    "torch::autograd::collect_next_edges",
    "torch::autograd::create_gradient_edge",
    "torch::autograd::deleteNode",
    "torch::autograd::extract_vars",
    "torch::autograd::get_current_node",
    "torch::autograd::to_optional",
    "torch::autograd::to_output_type",
    "torch::nn::parallel::replicate",
    "torch::nn::parallel::parallel_apply",
    "torch::nn::parallel::data_parallel",
    "torch::python::add_module_bindings",
    "torch::python::bind_module",
    "torch::python::init_bindings",
    # at::native cuDNN internals
    "at::native::dataSize",
    "at::native::fixSizeOneDimStride",
    "at::native::operator<<",
    "at::native::getCudnnDataTypeFromScalarType",
    # c10 cuda pool functions (internal)
    "c10::cuda::getStreamFromPool",
    "c10::cuda::getStreamFromExternal",
    "c10::xpu::getStreamFromPool",
    "c10::xpu::getStreamFromExternal",
    # c10 private use backend registration (internal)
    "c10::get_privateuse1_backend",
    "c10::is_privateuse1_backend_registered",
    "c10::register_privateuse1_backend",
    "c10::isValidDeviceType",
    "c10::DeviceTypeName",
    # torch::stable::detail internals
    "torch::stable::detail::unbox_to_tuple_impl",
    "torch::stable::detail::unbox_to_tuple",
    "torch::stable::detail::box_from_tuple_impl",
    "torch::stable::detail::box_from_tuple",
    # torch::stable::accelerator (documented in stable API page)
    "torch::stable::accelerator::getCurrentStream",
}
```
- EN: This segment defines or updates `EXCLUDED_SYMBOLS`, which shapes later behavior in the file. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段定义或更新了 `EXCLUDED_SYMBOLS`，从而影响文件后续行为。 它会与 PyTorch 模块、符号或示例交互。

### Lines 278-279 / 第 278-279 行
```python

# Namespaces whose free functions should be checked for documentation
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。

### Lines 280-299 / 第 280-299 行
```python
PUBLIC_FUNCTION_NAMESPACES = {
    "torch",
    "torch::autograd",
    "torch::cuda",
    "torch::mps",
    "torch::xpu",
    "torch::fft",
    "torch::special",
    "torch::nn::functional",
    "torch::nn::init",
    "torch::nn::utils",
    "torch::nn::utils::rnn",
    "torch::data",
    "torch::stable",
    "torch::stable::accelerator",
    "c10",
    "c10::cuda",
    "c10::xpu",
    "at::cuda",
}
```
- EN: This segment defines or updates `PUBLIC_FUNCTION_NAMESPACES`, which shapes later behavior in the file. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段定义或更新了 `PUBLIC_FUNCTION_NAMESPACES`，从而影响文件后续行为。 它会与 PyTorch 模块、符号或示例交互。

### Lines 300-304 / 第 300-304 行
```python


# ─── XML parsing ─────────────────────────────────────────────────────────────
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。

### Lines 305-314 / 第 305-314 行
```python
def _is_excluded(symbol: str) -> bool:
    """Check if a symbol should be excluded from coverage tracking."""
    if symbol in INCLUDED_SYMBOLS:
        return False
    if symbol in EXCLUDED_SYMBOLS:
        return True
    for pattern in EXCLUDED_PATTERNS:
        if re.fullmatch(pattern, symbol):
            return True
    return False
```
- EN: This segment defines function `_is_excluded(symbol)` and packages a reusable step in the documentation workflow. The docstring summarizes its intent as: Check if a symbol should be excluded from coverage tracking. Notable calls include `re.fullmatch`, which hint at the services this function relies on. It uses pattern matching to transform or inspect text.
- CN: 这一段定义了函数 `_is_excluded(symbol)`，用于封装文档流程中的可复用步骤。 其文档字符串将意图概括为：Check if a symbol should be excluded from coverage tracking. 值得注意的调用包括 `re.fullmatch`，这些调用揭示了该函数依赖的服务。 它使用模式匹配来转换或检查文本。

### Lines 317-359 / 第 317-359 行
```python
def _categorize(name: str) -> str:
    """Assign a category based on the symbol's namespace."""
    if name.startswith("torch::nn::functional::"):
        return "torch::nn::functional"
    if name.startswith("torch::nn::init::"):
        return "torch::nn::init"
    if name.startswith("torch::nn::utils::"):
        return "torch::nn::utils"
    if name.startswith("torch::nn::"):
        # Distinguish modules from other nn symbols
        short = name.split("::")[-1]
        if short[0].isupper():
            return "torch::nn (modules)"
        return "torch::nn"
    if name.startswith("torch::optim::"):
        return "torch::optim"
    if name.startswith("torch::data::"):
        return "torch::data"
    if name.startswith("torch::autograd::"):
        return "torch::autograd"
    if name.startswith("torch::serialize::") or name in ("torch::save", "torch::load"):
        return "torch::serialize"
    if name.startswith("torch::stable::"):
        return "torch::stable"
    if name.startswith("torch::fft::"):
        return "torch::fft"
    if name.startswith("torch::special::"):
        return "torch::special"
    if name.startswith(("torch::cuda::", "torch::mps::", "torch::xpu::")):
        return "torch (device)"
    if name.startswith("torch::"):
        return "torch (core)"
    if name.startswith("c10::cuda::"):
        return "c10::cuda"
    if name.startswith("c10::xpu::"):
        return "c10::xpu"
    if name.startswith("c10::"):
        return "c10"
    if name.startswith("at::cuda::"):
        return "at::cuda"
    if name.startswith("at::"):
        return "at"
    return "other"
```
- EN: This segment defines function `_categorize(name)` and packages a reusable step in the documentation workflow. The docstring summarizes its intent as: Assign a category based on the symbol's namespace. Notable calls include `name.startswith`, `short[0].isupper`, `name.split`, which hint at the services this function relies on. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段定义了函数 `_categorize(name)`，用于封装文档流程中的可复用步骤。 其文档字符串将意图概括为：Assign a category based on the symbol's namespace. 值得注意的调用包括 `name.startswith`, `short[0].isupper`, `name.split`，这些调用揭示了该函数依赖的服务。 它会与 PyTorch 模块、符号或示例交互。

### Lines 362-430 / 第 362-430 行
```python
def discover_apis_from_xml(xml_dir: Path) -> dict[str, list[tuple[str, str]]]:
    """Parse Doxygen index.xml to discover all public APIs.

    Returns dict of category -> list of (symbol, kind).
    """
    index_path = xml_dir / "index.xml"
    if not index_path.exists():
        print(
            f"ERROR: {index_path} not found. Run 'make doxygen' first.",
            file=sys.stderr,
        )
        sys.exit(1)

    tree = ET.parse(index_path)
    root = tree.getroot()

    apis: dict[str, list[tuple[str, str]]] = {}

    # Collect classes and structs
    for compound in root.findall("compound"):
        kind = compound.get("kind")
        if kind not in ("class", "struct"):
            continue
        name = compound.find("name").text
        if _is_excluded(name):
            continue
        category = _categorize(name)
        apis.setdefault(category, []).append((name, kind))

    # Collect free functions from public namespaces
    for compound in root.findall("compound"):
        if compound.get("kind") != "namespace":
            continue
        ns_name = compound.find("name").text
        if ns_name not in PUBLIC_FUNCTION_NAMESPACES:
            continue
        seen_funcs = set()
        for member in compound.findall("member"):
            if member.get("kind") != "function":
                continue
            func_name = member.find("name").text
            qualified = f"{ns_name}::{func_name}"
            if qualified in seen_funcs:
                continue  # skip overloads
            seen_funcs.add(qualified)
            if _is_excluded(qualified):
                continue
            category = _categorize(qualified)
            apis.setdefault(category, []).append((qualified, "function"))

    # Collect macros (defines) from file compounds
    for compound in root.findall("compound"):
        if compound.get("kind") != "file":
            continue
        for member in compound.findall("member"):
            if member.get("kind") != "define":
                continue
            macro_name = member.find("name").text
            # Only track well-known public macros
            if macro_name.startswith(("TORCH_LIBRARY", "TORCH_MODULE")):
                if _is_excluded(macro_name):
                    continue
                apis.setdefault("torch (macros)", []).append((macro_name, "define"))

    # Sort each category and deduplicate
    for category in apis:
        apis[category] = sorted(set(apis[category]))

    return apis
```
- EN: This segment defines function `discover_apis_from_xml(xml_dir)` and packages a reusable step in the documentation workflow. The docstring summarizes its intent as: Parse Doxygen index.xml to discover all public APIs. Returns dict of category -> list of (symbol, kind). Notable calls include `ET.parse`, `tree.getroot`, `root.findall`, `index_path.exists`, `print`, `sys.exit`, which hint at the services this function relies on. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段定义了函数 `discover_apis_from_xml(xml_dir)`，用于封装文档流程中的可复用步骤。 其文档字符串将意图概括为：Parse Doxygen index.xml to discover all public APIs. Returns dict of category -> list of (symbol, kind). 值得注意的调用包括 `ET.parse`, `tree.getroot`, `root.findall`, `index_path.exists`, `print`, `sys.exit`，这些调用揭示了该函数依赖的服务。 它会与 PyTorch 模块、符号或示例交互。

### Lines 431-435 / 第 431-435 行
```python


# ─── Source scanning ─────────────────────────────────────────────────────────

# RST directives: .. doxygenclass:: torch::nn::ReLU
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。 它会与 PyTorch 模块、符号或示例交互。

### Lines 436-440 / 第 436-440 行
```python
RST_DIRECTIVE_RE = re.compile(
    r"^\.\.\s+doxygen(class|struct|function|typedef|define|enum|namespace)"
    r"::\s*(.+?)\s*$",
    re.MULTILINE,
)
```
- EN: This segment defines or updates `RST_DIRECTIVE_RE`, which shapes later behavior in the file. It configures or cooperates with Sphinx documentation machinery. It uses pattern matching to transform or inspect text.
- CN: 这一段定义或更新了 `RST_DIRECTIVE_RE`，从而影响文件后续行为。 它会配置或配合 Sphinx 文档机制。 它使用模式匹配来转换或检查文本。

### Lines 442-445 / 第 442-445 行
```python
RST_CPP_DIRECTIVE_RE = re.compile(
    r"^\.\.\s+cpp:(class|struct|function|enum|type)" r"::\s*(.+?)\s*$",
    re.MULTILINE,
)
```
- EN: This segment defines or updates `RST_CPP_DIRECTIVE_RE`, which shapes later behavior in the file. It configures or cooperates with Sphinx documentation machinery. It uses pattern matching to transform or inspect text.
- CN: 这一段定义或更新了 `RST_CPP_DIRECTIVE_RE`，从而影响文件后续行为。 它会配置或配合 Sphinx 文档机制。 它使用模式匹配来转换或检查文本。

### Lines 446-447 / 第 446-447 行
```python

# MyST directives: ```{doxygenclass} torch::nn::ReLU
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。 它会与 PyTorch 模块、符号或示例交互。

### Lines 448-451 / 第 448-451 行
```python
MYST_DIRECTIVE_RE = re.compile(
    r"^`{3,}\{doxygen(class|struct|function|typedef|define|enum|namespace)\}\s*(.+?)\s*$",
    re.MULTILINE,
)
```
- EN: This segment defines or updates `MYST_DIRECTIVE_RE`, which shapes later behavior in the file. It uses pattern matching to transform or inspect text.
- CN: 这一段定义或更新了 `MYST_DIRECTIVE_RE`，从而影响文件后续行为。 它使用模式匹配来转换或检查文本。

### Lines 453-456 / 第 453-456 行
```python
MYST_CPP_DIRECTIVE_RE = re.compile(
    r"^`{3,}\{cpp:(class|struct|function|enum|type)\}\s*(.+?)\s*$",
    re.MULTILINE,
)
```
- EN: This segment defines or updates `MYST_CPP_DIRECTIVE_RE`, which shapes later behavior in the file. It uses pattern matching to transform or inspect text.
- CN: 这一段定义或更新了 `MYST_CPP_DIRECTIVE_RE`，从而影响文件后续行为。 它使用模式匹配来转换或检查文本。

### Lines 459-483 / 第 459-483 行
```python
def scan_sources(source_dir: Path) -> set[str]:
    """Extract all documented symbols from RST/MyST breathe and cpp domain directives."""
    documented = set()
    for src_file in list(source_dir.rglob("*.rst")) + list(source_dir.rglob("*.md")):
        content = src_file.read_text(errors="replace")
        patterns = (
            RST_DIRECTIVE_RE,
            RST_CPP_DIRECTIVE_RE,
            MYST_DIRECTIVE_RE,
            MYST_CPP_DIRECTIVE_RE,
        )
        for pattern in patterns:
            for match in pattern.finditer(content):
                symbol = match.group(2)
                # Strip template prefix
                if symbol.startswith("template"):
                    gt = symbol.find(">")
                    if gt != -1:
                        symbol = symbol[gt + 1 :].lstrip()
                # Strip function signature
                paren = symbol.find("(")
                if paren != -1:
                    symbol = symbol[:paren].rstrip()
                documented.add(symbol)
    return documented
```
- EN: This segment defines function `scan_sources(source_dir)` and packages a reusable step in the documentation workflow. The docstring summarizes its intent as: Extract all documented symbols from RST/MyST breathe and cpp domain directives. Notable calls include `set`, `list`, `src_file.read_text`, `source_dir.rglob`, `pattern.finditer`, `match.group`, which hint at the services this function relies on. It configures or cooperates with Sphinx documentation machinery.
- CN: 这一段定义了函数 `scan_sources(source_dir)`，用于封装文档流程中的可复用步骤。 其文档字符串将意图概括为：Extract all documented symbols from RST/MyST breathe and cpp domain directives. 值得注意的调用包括 `set`, `list`, `src_file.read_text`, `source_dir.rglob`, `pattern.finditer`, `match.group`，这些调用揭示了该函数依赖的服务。 它会配置或配合 Sphinx 文档机制。

### Lines 484-488 / 第 484-488 行
```python


# ─── Coverage report ─────────────────────────────────────────────────────────
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。

### Lines 489-545 / 第 489-545 行
```python
def generate_coverage_report(
    apis: dict[str, list[tuple[str, str]]], documented: set[str]
) -> str:
    """Generate a coverage report comparing discovered APIs against RST docs."""
    lines = []
    lines.append("Undocumented C++ objects")
    lines.append("=" * 50)
    lines.append("")

    total = 0
    total_missing = 0
    section_stats = []

    for category in sorted(apis.keys()):
        symbols = apis[category]
        section_missing = []
        for symbol, kind in symbols:
            total += 1
            unqualified = symbol.rsplit("::", 1)[-1]
            if symbol not in documented and unqualified not in documented:
                section_missing.append((symbol, kind))
                total_missing += 1

        covered = len(symbols) - len(section_missing)
        section_stats.append((category, covered, len(symbols)))

        if section_missing:
            lines.append(category)
            lines.append("-" * len(category))
            for symbol, kind in section_missing:
                lines.append(f"   * {symbol}  ({kind})")
            lines.append("")

    # Summary
    total_covered = total - total_missing
    pct = (total_covered / total * 100) if total else 0

    lines.append("")
    lines.append("=" * 50)
    lines.append("Summary")
    lines.append("=" * 50)
    lines.append("")
    lines.append(f"Total APIs discovered:   {total}")
    lines.append(f"Documented:              {total_covered}")
    lines.append(f"Missing:                 {total_missing}")
    lines.append(f"Coverage:                {pct:.1f}%")
    lines.append("")

    # Per-section table
    lines.append(f"{'Category':<45} {'Covered':>8} {'Total':>6} {'%':>7}")
    lines.append("-" * 70)
    for category, covered, section_total in section_stats:
        spct = (covered / section_total * 100) if section_total else 0
        lines.append(f"{category:<45} {covered:>8} {section_total:>6} {spct:>6.1f}%")
    lines.append("")

    return "\n".join(lines)
```
- EN: This segment defines function `generate_coverage_report(apis, documented)` and packages a reusable step in the documentation workflow. The docstring summarizes its intent as: Generate a coverage report comparing discovered APIs against RST docs. Notable calls include `lines.append`, `sorted`, `'\n'.join`, `apis.keys`, `section_stats.append`, `len`, which hint at the services this function relies on.
- CN: 这一段定义了函数 `generate_coverage_report(apis, documented)`，用于封装文档流程中的可复用步骤。 其文档字符串将意图概括为：Generate a coverage report comparing discovered APIs against RST docs. 值得注意的调用包括 `lines.append`, `sorted`, `'\n'.join`, `apis.keys`, `section_stats.append`, `len`，这些调用揭示了该函数依赖的服务。

### Lines 546-549 / 第 546-549 行
```python


# ─── HTML checks ─────────────────────────────────────────────────────────────
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。

### Lines 550-571 / 第 550-571 行
```python
BROKEN_PATTERNS = [
    (
        re.compile(r"Cannot find (?:class|struct|function|file)", re.IGNORECASE),
        "unresolved breathe directive",
    ),
    (
        re.compile(r"Unable to resolve (?:class|struct|function)", re.IGNORECASE),
        "unresolved breathe directive (ambiguous overload)",
    ),
    (
        re.compile(r"doxygenclass:|doxygenfunction:|doxygenstruct:", re.IGNORECASE),
        "raw directive text in output",
    ),
    (
        re.compile(r"<span class=\"problematic\">", re.IGNORECASE),
        "Sphinx problematic node (broken reference)",
    ),
    (
        re.compile(r"System Message:", re.IGNORECASE),
        "Sphinx system message (build error)",
    ),
]
```
- EN: This segment defines or updates `BROKEN_PATTERNS`, which shapes later behavior in the file. It configures or cooperates with Sphinx documentation machinery. It uses pattern matching to transform or inspect text.
- CN: 这一段定义或更新了 `BROKEN_PATTERNS`，从而影响文件后续行为。 它会配置或配合 Sphinx 文档机制。 它使用模式匹配来转换或检查文本。

### Lines 573-573 / 第 573-573 行
```python
MIN_CONTENT_LENGTH = 500
```
- EN: This segment defines or updates `MIN_CONTENT_LENGTH`, which shapes later behavior in the file.
- CN: 这一段定义或更新了 `MIN_CONTENT_LENGTH`，从而影响文件后续行为。

### Lines 576-621 / 第 576-621 行
```python
def check_html_output(build_dir: Path) -> str:
    """Check built HTML for broken formatting and empty pages."""
    issues = []

    if not build_dir.exists():
        return "ERROR: build/html directory not found. Run 'make html' first.\n"

    for html_file in sorted(build_dir.rglob("*.html")):
        rel = html_file.relative_to(build_dir)
        if rel.name in ("search.html", "genindex.html", "objects.inv"):
            continue

        try:
            content = html_file.read_text(errors="replace")
        except Exception as e:
            issues.append((str(rel), f"cannot read: {e}"))
            continue

        for pattern, description in BROKEN_PATTERNS:
            matches = pattern.findall(content)
            if matches:
                issues.append((str(rel), f"{description} ({len(matches)}x)"))

        if str(rel).startswith("api/"):
            text = re.sub(r"<[^>]+>", "", content)
            text = re.sub(r"\s+", " ", text).strip()
            if len(text) < MIN_CONTENT_LENGTH:
                issues.append((str(rel), f"possibly empty page ({len(text)} chars)"))

    lines = []
    lines.append("HTML Formatting Check")
    lines.append("=" * 50)
    lines.append("")

    if not issues:
        lines.append("No issues found.")
    else:
        lines.append(f"Found {len(issues)} issue(s):")
        lines.append("")
        lines.append(f"{'File':<55} Issue")
        lines.append("-" * 90)
        for filepath, issue in issues:
            lines.append(f"{filepath:<55} {issue}")

    lines.append("")
    return "\n".join(lines)
```
- EN: This segment defines function `check_html_output(build_dir)` and packages a reusable step in the documentation workflow. The docstring summarizes its intent as: Check built HTML for broken formatting and empty pages. Notable calls include `sorted`, `lines.append`, `'\n'.join`, `build_dir.exists`, `build_dir.rglob`, `html_file.relative_to`, which hint at the services this function relies on. It uses pattern matching to transform or inspect text.
- CN: 这一段定义了函数 `check_html_output(build_dir)`，用于封装文档流程中的可复用步骤。 其文档字符串将意图概括为：Check built HTML for broken formatting and empty pages. 值得注意的调用包括 `sorted`, `lines.append`, `'\n'.join`, `build_dir.exists`, `build_dir.rglob`, `html_file.relative_to`，这些调用揭示了该函数依赖的服务。 它使用模式匹配来转换或检查文本。

### Lines 622-626 / 第 622-626 行
```python


# ─── coverxygen integration ─────────────────────────────────────────────────
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。

### Lines 627-706 / 第 627-706 行
```python
def run_coverxygen(xml_dir: Path) -> str:
    """Run coverxygen on Doxygen XML output for doc-comment coverage."""
    lines = []
    lines.append("Coverxygen Report (Doxygen doc-comment coverage)")
    lines.append("=" * 50)
    lines.append("")

    if not xml_dir.exists():
        lines.append("ERROR: build/xml directory not found. Run 'make doxygen' first.")
        return "\n".join(lines)

    coverxygen_cmd = None
    for cmd in [
        ["coverxygen", "--version"],
        [sys.executable, "-m", "coverxygen", "--version"],
    ]:
        try:
            subprocess.run(cmd, capture_output=True, check=True)
            coverxygen_cmd = cmd[:-1]
            break
        except (FileNotFoundError, subprocess.CalledProcessError):
            continue
    if coverxygen_cmd is None:
        lines.append("coverxygen not installed. Install with: pip install coverxygen")
        lines.append("")
        lines.append("Once installed, coverxygen analyzes Doxygen XML to report what")
        lines.append("percentage of C++ symbols have doc comments in the source code.")
        lines.append("This is complementary to the RST coverage check above.")
        lines.append("")
        lines.append("Usage:")
        lines.append(
            f"  coverxygen --xml-dir {xml_dir} --src-dir ../../ --output coverxygen.info"
        )
        lines.append("  # Then use lcov/genhtml to visualize:")
        lines.append(
            "  genhtml --no-function-coverage coverxygen.info -o coverxygen_html"
        )
        return "\n".join(lines)

    try:
        result = subprocess.run(
            coverxygen_cmd
            + [
                "--xml-dir",
                str(xml_dir),
                "--src-dir",
                str(SCRIPT_DIR / ".." / ".."),
                "--output",
                "-",
                "--kind",
                "class,struct,function",
                "--scope",
                "public",
                "--exclude",
                ".*/build/.*",
                "--exclude",
                ".*/detail/.*",
                "--exclude",
                ".*/nativert/.*",
                "--exclude",
                ".*/stable/library\\.h",
            ],
            capture_output=True,
            text=True,
            timeout=120,
        )
        if result.returncode == 0:
            total = 0
            documented_count = 0
            for line in result.stdout.splitlines():
                if line.startswith("DA:"):
                    total += 1
                    parts = line.split(",")
                    if len(parts) >= 2 and parts[1].strip() != "0":
                        documented_count += 1
            pct = (documented_count / total * 100) if total else 0
            lines.append(f"Symbols scanned:    {total}")
            lines.append(f"With doc comments:  {documented_count}")
            lines.append(f"Coverage:           {pct:.1f}%")
            lines.append("")
```
- EN: This is part 1/2 of function `run_coverxygen(xml_dir)`, so it continues the implementation rather than introducing a new top-level concept. The docstring summarizes its intent as: Run coverxygen on Doxygen XML output for doc-comment coverage. Notable calls include `lines.append`, `'\n'.join`, `xml_dir.exists`, `subprocess.run`, `result.stdout.splitlines`, `(SCRIPT_DIR / 'coverxygen.info').write_text`, which hint at the services this function relies on. It launches external commands or tools.
- CN: 这一段是函数 `run_coverxygen(xml_dir)` 的 1/2 部分，因此它是在继续实现，而不是引入新的顶层概念。 其文档字符串将意图概括为：Run coverxygen on Doxygen XML output for doc-comment coverage. 值得注意的调用包括 `lines.append`, `'\n'.join`, `xml_dir.exists`, `subprocess.run`, `result.stdout.splitlines`, `(SCRIPT_DIR / 'coverxygen.info').write_text`，这些调用揭示了该函数依赖的服务。 它会调用外部命令或工具。

### Lines 707-718 / 第 707-718 行
```python
            lines.append("Full lcov output saved to: coverxygen.info")
            (SCRIPT_DIR / "coverxygen.info").write_text(result.stdout)
        else:
            lines.append(f"coverxygen failed (exit {result.returncode}):")
            lines.append(result.stderr[:500])
    except subprocess.TimeoutExpired:
        lines.append("coverxygen timed out after 120s")
    except Exception as e:
        lines.append(f"coverxygen error: {e}")

    lines.append("")
    return "\n".join(lines)
```
- EN: This is part 2/2 of function `run_coverxygen(xml_dir)`, so it continues the implementation rather than introducing a new top-level concept. The docstring summarizes its intent as: Run coverxygen on Doxygen XML output for doc-comment coverage. Notable calls include `lines.append`, `'\n'.join`, `xml_dir.exists`, `subprocess.run`, `result.stdout.splitlines`, `(SCRIPT_DIR / 'coverxygen.info').write_text`, which hint at the services this function relies on. It launches external commands or tools.
- CN: 这一段是函数 `run_coverxygen(xml_dir)` 的 2/2 部分，因此它是在继续实现，而不是引入新的顶层概念。 其文档字符串将意图概括为：Run coverxygen on Doxygen XML output for doc-comment coverage. 值得注意的调用包括 `lines.append`, `'\n'.join`, `xml_dir.exists`, `subprocess.run`, `result.stdout.splitlines`, `(SCRIPT_DIR / 'coverxygen.info').write_text`，这些调用揭示了该函数依赖的服务。 它会调用外部命令或工具。

### Lines 719-723 / 第 719-723 行
```python


# ─── Main ────────────────────────────────────────────────────────────────────
```
- EN: This single segment contains top-level statements, comments, or continuation code outside a major AST node.
- CN: 这一段包含主要 AST 节点之外的顶层语句、注释或续接代码。

### Lines 724-778 / 第 724-778 行
```python
def main():
    parser = argparse.ArgumentParser(description="C++ docs coverage checker")
    parser.add_argument(
        "--coverxygen",
        action="store_true",
        help="Also run coverxygen on Doxygen XML for doc-comment coverage",
    )
    parser.add_argument(
        "--html-only",
        action="store_true",
        help="Only run HTML formatting checks",
    )
    args = parser.parse_args()

    reports = []

    if not args.html_only:
        # Phase 1: Discover APIs from Doxygen XML
        print("Discovering APIs from Doxygen XML...")
        apis = discover_apis_from_xml(BUILD_XML)
        total_apis = sum(len(v) for v in apis.values())
        print(f"  Found {total_apis} public APIs across {len(apis)} categories")

        # Phase 2: Scan RST for documented symbols
        print("Scanning sources for breathe directives...")
        documented = scan_sources(SOURCE_DIR)
        print(f"  Found {len(documented)} documented symbols")

        coverage_report = generate_coverage_report(apis, documented)
        reports.append(coverage_report)

        COVERAGE_OUTPUT.write_text(coverage_report)
        print(f"  Coverage report written to: {COVERAGE_OUTPUT}")

    # Phase 3: HTML checks
    print("Checking HTML output for formatting issues...")
    html_report = check_html_output(BUILD_HTML)
    reports.append(html_report)
    HTML_REPORT.write_text(html_report)
    print(f"  HTML report written to: {HTML_REPORT}")

    # Phase 4: coverxygen (optional)
    if args.coverxygen:
        print("Running coverxygen...")
        cov_report = run_coverxygen(BUILD_XML)
        reports.append(cov_report)

    # Print everything
    print()
    print("=" * 60)
    for report in reports:
        print(report)
        print()

    return 0
```
- EN: This segment defines function `main()` and packages a reusable step in the documentation workflow. Notable calls include `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `check_html_output`, `reports.append`, which hint at the services this function relies on. It participates in command-line argument parsing.
- CN: 这一段定义了函数 `main()`，用于封装文档流程中的可复用步骤。 值得注意的调用包括 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `check_html_output`, `reports.append`，这些调用揭示了该函数依赖的服务。 它参与命令行参数解析。

### Lines 781-782 / 第 781-782 行
```python
if __name__ == "__main__":
    sys.exit(main())
```
- EN: This conditional branch evaluates `__name__ == '__main__'` and gates behavior on that runtime or configuration check.
- CN: 这一条件分支会评估 `__name__ == '__main__'`，并据此控制相应的运行时或配置行为。

## Key Concepts / 关键概念
- EN: `argparse` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `argparse` —— 在标题、示例或行内强调中反复出现的主题。
- EN: `re` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `re` —— 在标题、示例或行内强调中反复出现的主题。
- EN: `subprocess` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `subprocess` —— 在标题、示例或行内强调中反复出现的主题。
- EN: `sys` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `sys` —— 在标题、示例或行内强调中反复出现的主题。
- EN: `xml.etree.ElementTree` — API symbol or namespace repeatedly referenced by the file.
  CN: `xml.etree.ElementTree` —— 文件中反复引用的 API 符号或命名空间。
- EN: `pathlib.Path` — API symbol or namespace repeatedly referenced by the file.
  CN: `pathlib.Path` —— 文件中反复引用的 API 符号或命名空间。
- EN: `_is_excluded` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `_is_excluded` —— 在标题、示例或行内强调中反复出现的主题。
- EN: `_categorize` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `_categorize` —— 在标题、示例或行内强调中反复出现的主题。
- EN: `discover_apis_from_xml` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `discover_apis_from_xml` —— 在标题、示例或行内强调中反复出现的主题。
- EN: `scan_sources` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `scan_sources` —— 在标题、示例或行内强调中反复出现的主题。

## Dependencies / 依赖关系
- EN: Depends on `argparse` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `argparse`。
- EN: Depends on `re` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `re`。
- EN: Depends on `subprocess` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `subprocess`。
- EN: Depends on `sys` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `sys`。
- EN: Depends on `xml.etree.ElementTree` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `xml.etree.ElementTree`。
- EN: Depends on `pathlib.Path` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `pathlib.Path`。
- EN: Depends on `doxygen` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `doxygen`。

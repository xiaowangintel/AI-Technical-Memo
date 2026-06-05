# model_inspection.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_inspection.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: EN: Formats a PyTorch model into a readable tree that groups repeated numbered layers and surfaces quantization metadata. / CN: 把 PyTorch 模型格式化为可读的层级树，并在其中合并重复的编号层、展示量化相关元数据。

## Line-by-Line Analysis / 逐行分析

### Module info extraction
```python
def _get_module_info(module: nn.Module) -> str:
    """Get info string for a module."""
    class_name = type(module).__name__
    parts = []

    # Add quant_method if present
    quant_method = getattr(module, "quant_method", None)
    if quant_method is not None:
        quant_name = type(quant_method).__name__
        # For CompressedTensors, show the underlying scheme instead
        scheme = getattr(module, "scheme", None)
        if scheme is not None:
            quant_name = type(scheme).__name__
        # Skip unquantized methods
        if "Unquantized" not in quant_name:
            parts.append(f"quant={quant_name}")

    # If module has extra_repr, use it
    if hasattr(module, "extra_repr"):
        parts.append(module.extra_repr().replace("\n", ""))

    if parts:
        return f"{class_name}({', '.join(parts)})"

    # For unknown modules, use the default PyTorch repr
    return str(module)
```
**EN:** `_get_module_info()` builds the printable label for one module. It prefers explicit quantization metadata when present, falls back to `extra_repr()` for layer-specific details, and suppresses “Unquantized” markers so the output stays focused on meaningful differences.
**CN:** `_get_module_info()` 负责生成单个模块的可打印标签。它会优先读取显式的量化元数据，在可用时再拼接 `extra_repr()` 中的层特定信息，并主动跳过 “Unquantized” 之类的噪声标记，使输出更聚焦于真正有意义的差异。

### Child signatures and index compression
```python
def _get_child_signature(child: nn.Module) -> str:
    """Get a signature for a child module to detect duplicates."""
    lines = []
    for name, submodule in child.named_modules():
        lines.append(f"{name}:{_get_module_info(submodule)}")
    return "\n".join(lines)


def _format_index_ranges(indices: list[int]) -> str:
    """Format indices into range notation (e.g., [0,1,2,4,5,6] -> '0-2, 4-6')."""
    indices = sorted(indices)
    ranges = []
    start = end = indices[0]

    for idx in indices[1:]:
        if idx == end + 1:
            end = idx
        else:
            ranges.append(str(start) if start == end else f"{start}-{end}")
            start = end = idx

    ranges.append(str(start) if start == end else f"{start}-{end}")
    return ", ".join(ranges)
```
**EN:** `_get_child_signature()` recursively serializes a child module's structure into a comparable string. `_format_index_ranges()` then turns sorted numeric indices into compact ranges such as `0-2, 4-6`, which is what allows the tree formatter to collapse many identical layers into one summary block.
**CN:** `_get_child_signature()` 会把子模块的结构递归序列化成可比较的字符串。随后 `_format_index_ranges()` 会把排序后的数字索引压缩成 `0-2, 4-6` 这样的区间表示，这正是后续树形格式化器能够把大量相同层折叠成一个摘要块的基础。

### Recursive module tree formatter
```python
def _format_module_tree(
    module: nn.Module,
    name: str = "",
    indent: int = 0,
) -> list[str]:
    """Format a module tree with indentation, grouping identical layers.

    Produces output like:
        (layers): ModuleList(
          (0-27, 29-47): 47 x LlamaDecoderLayer(
            ...
          )
          (28, 48): 2 x DifferentDecoderLayer(
            ...
          )
        )
    """
    lines = []
    prefix = "  " * indent
    children = list(module.named_children())

    # Leaf node - just output the module info
    if not children:
        info = _get_module_info(module)
        lines.append(f"{prefix}({name}): {info}" if name else f"{prefix}{info}")
        return lines

    # Non-leaf node - output opening line and recurse into children
    info = _get_module_info(module)
    lines.append(f"{prefix}({name}): {info}(" if name else f"{prefix}{info}(")

    # Separate numbered children (e.g., "0", "1") from named ones (e.g., "norm")
    numbered: list[tuple[int, nn.Module]] = []
    non_numbered: list[tuple[str, nn.Module]] = []
    for child_name, child_module in children:
        try:
            numbered.append((int(child_name), child_module))
        except ValueError:
            non_numbered.append((child_name, child_module))

    # Group numbered children by structure signature to collapse identical layers
    # e.g., layers 0-27 and 29-47 with same structure become "(0-27, 29-47): 47 x"
    if numbered:
        sig_to_group: dict[str, list[tuple[int, nn.Module]]] = {}
        for idx, child_module in numbered:
            sig = _get_child_signature(child_module)
            sig_to_group.setdefault(sig, []).append((idx, child_module))

        # Output groups sorted by first index
        for group in sorted(sig_to_group.values(), key=lambda g: g[0][0]):
            indices = [idx for idx, _ in group]
            representative = group[0][1]
            child_lines = _format_module_tree(representative, "", indent + 1)
            first_line = child_lines[0].lstrip()
            child_prefix = "  " * (indent + 1)

            if len(indices) > 1:
                range_str = _format_index_ranges(indices)
                child_lines[0] = (
                    f"{child_prefix}({range_str}): {len(indices)} x {first_line}"
                )
            else:
                child_lines[0] = f"{child_prefix}({indices[0]}): {first_line}"
            lines.extend(child_lines)

    # Output non-numbered children (e.g., "embed_tokens", "norm")
    for child_name, child_module in non_numbered:
        lines.extend(_format_module_tree(child_module, child_name, indent + 1))

    lines.append(f"{prefix})")
    return lines
```
**EN:** `_format_module_tree()` is the core formatter. Leaf nodes print a single line, while non-leaf nodes recurse with indentation; numbered children are grouped by structural signature so repeated transformer blocks show up as `N x Layer(...)`, whereas named children like `norm` or `embed_tokens` remain individually visible.
**CN:** `_format_module_tree()` 是核心格式化逻辑。叶子节点只输出一行；非叶子节点会带缩进递归展开；编号子节点会按结构签名分组，因此重复的 transformer block 会显示成 `N x Layer(...)`，而 `norm`、`embed_tokens` 这类具名子节点仍会单独展示。

### Public formatting entry point
```python
def format_model_inspection(model: nn.Module) -> str:
    """Format a model into a transformers-style hierarchical string."""
    return "\n".join(_format_module_tree(model))
```
**EN:** `format_model_inspection()` is intentionally thin: it simply joins the recursive formatter's lines into the final human-readable string. That separation keeps the tree construction reusable and easy to test.
**CN:** `format_model_inspection()` 设计得很薄，只负责把递归格式化器生成的多行结果拼接成最终字符串。这样的拆分让树构建逻辑更易复用，也更便于测试。

## Key Concepts / 关键概念
- **Structural deduplication** — EN: Repeated numbered layers are collapsed by signature instead of printed one by one. / CN: 重复的编号层会按结构签名折叠，而不是逐个展开打印。
- **Quantization-aware inspection** — EN: The formatter surfaces quantization schemes so deployment-specific structure stays visible. / CN: 格式化结果会显式展示量化方案，便于观察部署相关的结构差异。
- **Transformers-style tree output** — EN: The resulting string resembles familiar PyTorch/Transformers module dumps. / CN: 最终字符串风格接近大家熟悉的 PyTorch / Transformers 模块树输出。

## Dependencies / 依赖关系
- **torch.nn** — EN: Supplies the `nn.Module` abstraction traversed by the formatter. / CN: 提供格式化器要遍历的 `nn.Module` 抽象。
- **Module.extra_repr()** — EN: Layer-specific repr fragments are reused to enrich the printed tree. / CN: 层自身的 `extra_repr()` 片段会被复用，以增强输出细节。
- **Quantized modules** — EN: Optional `quant_method` and `scheme` attributes are inspected when available. / CN: 若模块存在 `quant_method` 和 `scheme` 属性，代码会一并检查并展示。

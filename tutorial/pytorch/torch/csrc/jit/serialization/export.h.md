# export.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/serialization/export.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
#pragma once

#include <caffe2/serialize/inline_container.h>
#include <torch/csrc/jit/api/module.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/serialization/export_bytecode.h>
#include <torch/csrc/jit/serialization/flatbuffer_serializer.h>
#include <torch/csrc/jit/serialization/python_print.h>
#include <torch/csrc/jit/serialization/storage_context.h>
#include <torch/csrc/jit/serialization/type_name_uniquer.h>
#include <torch/csrc/onnx/onnx.h>
#include <ostream>
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/api/module.h, torch/csrc/jit/ir/ir.h, torch/csrc/jit/serialization/export_bytecode.h, and 5 more; standard-library headers such as caffe2/serialize/inline_container.h, ostream. The preprocessor guard keeps declarations single-instanced when the header is included transitively.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/api/module.h、torch/csrc/jit/ir/ir.h、torch/csrc/jit/serialization/export_bytecode.h 等共 8 项；标准库头文件，如 caffe2/serialize/inline_container.h、ostream。 预处理器保护用于避免头文件在传递包含时被重复展开。

### Lines 14-28
```cpp
namespace ONNX_NAMESPACE {
class ModelProto;
}

namespace torch::jit {

// This map is used to keep track of parameters that should be exported
// externally. When `defer_weight_export` is true, the returned map contains
// kv pairs that map {external reference name} -> {at::Tensor to be exported}.
// It is the responsibility of the caller to export these appropriately.
//
// For example, when exporting to a zip archive, the caller may write out files
// for each entry in the export map, with the filename being the key and the
// file contents being the raw tensor data.
using RawDataExportMap = std::unordered_map<std::string, at::Tensor>;
```
- **EN**: The namespace declarations place the code inside ONNX_NAMESPACE, torch::jit, matching the surrounding JIT subsystem. It introduces or extends ModelProto, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 命名空间声明把代码放入 ONNX_NAMESPACE、torch::jit 中，与周边 JIT 子系统保持一致。 它引入或扩展了 ModelProto，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 30-45
```cpp
using SymbolDimMap = std::map<c10::ShapeSymbol, std::string>;
using DimSymbolMap = std::map<std::string, c10::ShapeSymbol>;

using NodeNameMap = std::unordered_map<const Node*, std::string>;

// Used for modularized export settling function and node attributes.
using NodeAttrNameMap = std::
    unordered_map<const Node*, std::unordered_map<std::string, std::string>>;

TORCH_API std::tuple<
    std::shared_ptr<::ONNX_NAMESPACE::ModelProto>,
    RawDataExportMap,
    SymbolDimMap,
    bool,
    NodeNameMap>
export_onnx(
```
- **EN**: This chunk continues `ModelProto` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `ModelProto`，进一步展开其内部控制流或数据流转。

### Lines 46-61
```cpp
    const std::shared_ptr<Graph>& graph,
    const std::map<std::string, at::Tensor>& initializers,
    int64_t onnx_opset_version,
    const std::unordered_map<
        std::string,
        std::unordered_map<int64_t, std::string>>& dynamic_axes,
    bool defer_weight_export = false,
    ::torch::onnx::OperatorExportTypes operator_export_type =
        ::torch::onnx::OperatorExportTypes::ONNX,
    bool strip_doc_string = true,
    bool keep_initializers_as_inputs = true,
    const std::map<std::string, int>& custom_opsets = {},
    bool add_node_names = true,
    bool use_external_data_format = false,
    const std::string& onnx_file_path = std::string(),
    const NodeAttrNameMap& node_attr_to_name = {});
```
- **EN**: This chunk continues `ModelProto` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `ModelProto`，进一步展开其内部控制流或数据流转。

### Lines 63-73
```cpp
TORCH_API std::string serialize_model_proto_to_string(
    const std::shared_ptr<::ONNX_NAMESPACE::ModelProto>& model_proto);

TORCH_API void check_onnx_proto(const std::string& proto_string);

// Serializer for both oldsyle and unified format TorchScript serialization
class TORCH_API ScriptModuleSerializer {
 public:
  explicit ScriptModuleSerializer(
      caffe2::serialize::PyTorchStreamWriter& export_writer)
      : writer_(export_writer) {}
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `ScriptModuleSerializer`, which implements a focused step in loading or storing scripted programs.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `ScriptModuleSerializer`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。

### Lines 75-84
```cpp
  void writeFiles(const std::string& code_dir);
  void serialize(
      const Module& module,
      const ExtraFilesMap& extra_files,
      bool bytecode_format,
      bool save_mobile_debug_info);
  void serialize_unified_format(Module& module, uint64_t script_module_id);
  SerializationStorageContext& storage_context();

  ~ScriptModuleSerializer() = default;
```
- **EN**: This chunk declares `storage_context`, which implements a focused step in loading or storing scripted programs.
- **CN**: 这一段声明了 `storage_context`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。

### Lines 86-98
```cpp
 private:
  void convertNamedType(const c10::NamedTypePtr& class_type);
  void convertTypes(const at::NamedTypePtr& root_type);
  void writeExtraFiles(const Module& module, const ExtraFilesMap& extra_files);
  void writeByteCode(const Module& module, bool save_mobile_debug_info);
  void writeArchive(
      const IValue& value,
      const std::string& archive_name,
      const std::string& archive_dir,
      const std::string& tensor_dir,
      bool use_storage_context = false,
      bool skip_tensor_data = false);
  void updateSourceRangeTags(const SourceRangeRecords& ranges);
```
- **EN**: This chunk declares `updateSourceRangeTags`, which implements a focused step in loading or storing scripted programs.
- **CN**: 这一段声明了 `updateSourceRangeTags`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。

### Lines 100-113
```cpp
  caffe2::serialize::PyTorchStreamWriter& writer_;
  std::vector<at::IValue> constant_table_;

  std::unordered_set<c10::NamedTypePtr> converted_types_;
  PrintDepsTable class_deps_;
  TypeNameUniquer type_name_uniquer_;
  // qualifier, e.g. '__torch__.Bar' -> PythonPrint for the file that will be
  // created
  OrderedDict<std::string, PythonPrint> file_streams_;
  // Used to keep references of storages around during serialization to solve
  // for ABA memory reuse problem hit when storages are created/destroyed
  // during serialization process. Also used to coordinate sharing of storages
  // between Script and eager modules in torch.package.
  SerializationStorageContext storage_context_;
```
- **EN**: This chunk continues `updateSourceRangeTags` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `updateSourceRangeTags`，进一步展开其内部控制流或数据流转。

### Lines 115-130
```cpp
  // Uniquely identifies a SourceRange in a model.
  // SourceRanges are associated with Nodes of Graphs.
  // However for mobile deployment we dont intend to ship
  // full JIT with capabilities of reading code and constructing
  // graphs.
  // Instead we serialize the Code generated from graph of the methods.
  // Code is serialized in bytecode format that contains instructions
  // corresponding to the nodes of the graph. Since original graph is gone, the
  // question is how do we identify where the ops, in serialized bytecode, come
  // from in original model code. We do this in two parts.
  // 1. Associate a unique tag to SourceRange.
  // 2. Serialize this unique_tag.
  //  2.1 Meaning save <byte_offset, source_range_tag, source range> instead of
  //      <byte_offset, source range>
  // 3. During serializing model for mobile, i.e. bytecode generation,
  //    save unique tag of SourceRange corresponding to the Node.
```
- **EN**: This comment block documents assumptions, constraints, or generated-code provenance that shape the implementation below.
- **CN**: 这一段注释说明了后续实现依赖的假设、约束或生成来源。

### Lines 131-140
```cpp
  // 4. During deserialization, read all the debug_pkl, to construct a map
  //    of <unique_tag, SourceRange> and use tag saved with OPs in bytecode
  //    to lookup the source range.
  // Strictly speaking we will serialize InlinedCallStack directly, which
  // contains SourceRange. This way we have access to entire callstack and not
  // just source information about where the node is, since bytecode inlines the
  // graph before saving it.
  SourceRangeTagMap source_range_tags_;
  int64_t current_source_range_tag_{0};
};
```
- **EN**: This chunk continues `updateSourceRangeTags` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `updateSourceRangeTags`，进一步展开其内部控制流或数据流转。

### Lines 142-153
```cpp
// For testing purposes
TORCH_API std::string pretty_print_onnx(
    const std::shared_ptr<Graph>& graph,
    const std::map<std::string, at::Tensor>& initializers,
    int64_t onnx_opset_version,
    bool defer_weight_export,
    ::torch::onnx::OperatorExportTypes operator_export_type =
        ::torch::onnx::OperatorExportTypes::ONNX,
    bool google_printer = false,
    bool keep_initializers_as_inputs = true,
    const std::map<std::string, int>& custom_opsets = {},
    bool add_node_names = true);
```
- **EN**: This chunk continues `updateSourceRangeTags` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `updateSourceRangeTags`，进一步展开其内部控制流或数据流转。

### Lines 155-169
```cpp
TORCH_API void ExportModule(
    const Module& module,
    std::ostream& out,
    const ExtraFilesMap& metadata = ExtraFilesMap(),
    bool bytecode_format = false,
    bool save_mobile_debug_info = false,
    bool use_flatbuffer = false);

TORCH_API void ExportModule(
    const Module& module,
    const std::string& filename,
    const ExtraFilesMap& metadata = ExtraFilesMap(),
    bool bytecode_format = false,
    bool save_mobile_debug_info = false,
    bool use_flatbuffer = false);
```
- **EN**: This chunk continues `updateSourceRangeTags` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `updateSourceRangeTags`，进一步展开其内部控制流或数据流转。

### Lines 171-186
```cpp
TORCH_API void ExportModule(
    const Module& module,
    const std::function<size_t(const void*, size_t)>& writer_func,
    const ExtraFilesMap& metadata = ExtraFilesMap(),
    bool bytecode_format = false,
    bool save_mobile_debug_info = false,
    bool use_flatbuffer = false);

// Write the bytes of a pickle archive and the tensors referenced inside that
// archive
TORCH_API void writeArchiveAndTensors(
    const std::string& archive_name,
    const char* pickle_bytes,
    size_t size,
    const std::vector<at::Tensor>& tensors,
    caffe2::serialize::PyTorchStreamWriter& out);
```
- **EN**: This chunk declares `writeArchiveAndTensors`, which implements a focused step in loading or storing scripted programs.
- **CN**: 这一段声明了 `writeArchiveAndTensors`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。

### Lines 188-199
```cpp
// Surrounding system can install an additional hook to produce extra files
// with metadata based on environment every time a module is serialized.
using ExportModuleExtraFilesHook = std::function<ExtraFilesMap(const Module&)>;
TORCH_API void SetExportModuleExtraFilesHook(ExportModuleExtraFilesHook hook);

/**
 * Generates new bytecode for a Script module and returns what the op list
 * would be for a LiteScriptModule based off the current code base. If you
 * have a LiteScriptModule and want to get the currently present
 * list of ops call _export_operator_list instead.
 */
TORCH_API std::vector<std::string> export_opnames(const Module& m);
```
- **EN**: This chunk declares `export_opnames`, which exports runtime state into a serialized or portable representation.
- **CN**: 这一段声明了 `export_opnames`，其作用是把运行时状态导出为序列化或可移植表示。

### Lines 201-210
```cpp
struct TORCH_API BytecodeEmitMode {
  static bool is_default_value_for_unspecified_arg_enabled();
  static void set_default_value_for_unspecified_arg_enabled(bool enabled);

  static bool is_default_args_before_out_args_enabled();
  static void set_default_args_before_out_args_enabled(bool enabled);

  static bool is_emit_promoted_ops_enabled();
  static void set_default_emit_promoted_ops_enabled(bool enabled);
};
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file. This chunk defines `set_default_emit_promoted_ops_enabled`, which implements a focused step in loading or storing scripted programs.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `set_default_emit_promoted_ops_enabled`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。

### Lines 212-227
```cpp
// RAII guard to switch the way JIT emits the bytecode for inputs.
// default_value_for_unspecified_arg:
// true: instruction of default argument values (like LOADC) is emitted.
// false: instruction of default argument values are not emitted. Instead
// they are fetched from operator schema.
// default_args_before_out_args (to forward compatible support
// operators allowing out arguments and default arguments):
// true: the number of specified arguments will deserialized to (#all_args -
// #default_args). false: the number of specified arguments will deserialized to
// (#all_args).
struct TORCH_API BytecodeEmitModeGuard {
  BytecodeEmitModeGuard(
      bool enable_default_value_for_unspecified_arg,
      bool enable_default_args_before_out_args,
      bool enable_emit_promoted_ops)
      : prev_default_value_for_unspecified_arg_mode(
```
- **EN**: It introduces or extends TORCH_API, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 228-243
```cpp
            BytecodeEmitMode::is_default_value_for_unspecified_arg_enabled()),
        prev_default_args_before_out_args(
            BytecodeEmitMode::is_default_args_before_out_args_enabled()),
        prev_default_emit_promoted_ops(
            BytecodeEmitMode::is_emit_promoted_ops_enabled()) {
    BytecodeEmitMode::set_default_value_for_unspecified_arg_enabled(
        enable_default_value_for_unspecified_arg);
    BytecodeEmitMode::set_default_args_before_out_args_enabled(
        enable_default_args_before_out_args);
    BytecodeEmitMode::set_default_emit_promoted_ops_enabled(
        enable_emit_promoted_ops);
  }
  ~BytecodeEmitModeGuard() {
    BytecodeEmitMode::set_default_value_for_unspecified_arg_enabled(
        prev_default_value_for_unspecified_arg_mode);
    BytecodeEmitMode::set_default_args_before_out_args_enabled(
```
- **EN**: This chunk continues `TORCH_API` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `TORCH_API`，进一步展开其内部控制流或数据流转。

### Lines 244-255
```cpp
        prev_default_args_before_out_args);
    BytecodeEmitMode::set_default_emit_promoted_ops_enabled(
        prev_default_emit_promoted_ops);
  }
  bool prev_default_value_for_unspecified_arg_mode;
  bool prev_default_args_before_out_args;
  bool prev_default_emit_promoted_ops;
};

TORCH_API IValue to_tuple(std::vector<IValue> ivalues);
TORCH_API IValue
Table(const std::vector<std::pair<std::string, IValue>>& entries);
```
- **EN**: This chunk declares `to_tuple`, which implements a focused step in loading or storing scripted programs.
- **CN**: 这一段声明了 `to_tuple`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。

### Lines 257-270
```cpp
// TODO remove these switches once interface call is rolled out.
TORCH_API void enableMobileInterfaceCallExport();
bool getMobileInterfaceCallExport();

TORCH_API CompilationOptions getOptionsFromGlobal();

TORCH_API void save_jit_module(
    const Module& module,
    const std::string& filename,
    const ExtraFilesMap& extra_files = ExtraFilesMap());

TORCH_API DetachedBuffer::UniqueDetachedBuffer save_jit_module_to_bytes(
    const Module& module,
    const ExtraFilesMap& extra_files = ExtraFilesMap());
```
- **EN**: This chunk declares `save_jit_module_to_bytes`, which implements a focused step in loading or storing scripted programs.
- **CN**: 这一段声明了 `save_jit_module_to_bytes`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。

### Lines 272-278
```cpp
TORCH_API void save_jit_module_to_write_func(
    const Module& module,
    const ExtraFilesMap& extra_files,
    bool save_mobile_debug_info,
    const std::function<size_t(const void*, size_t)>& writer_func);

} // namespace torch::jit
```
- **EN**: This chunk declares `save_jit_module_to_write_func`, which implements a focused step in loading or storing scripted programs.
- **CN**: 这一段声明了 `save_jit_module_to_write_func`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。

## Key Concepts / 关键概念

- **Serialization pipeline**
  - EN: Implements TorchScript import/export, bytecode, source loading, and serialized model interchange.
  - CN: 实现 TorchScript 的导入/导出、字节码、源码加载以及序列化模型交换。
- **ModelProto**
  - EN: `ModelProto` is a central symbol declared or implemented in this file.
  - CN: `ModelProto` 是本文件声明或实现的核心符号。
- **TORCH_API**
  - EN: `TORCH_API` is a central symbol declared or implemented in this file.
  - CN: `TORCH_API` 是本文件声明或实现的核心符号。
- **Graph IR**
  - EN: Works on graph, node, and block structures that represent TorchScript programs.
  - CN: 围绕表示 TorchScript 程序的 graph、node 与 block 结构工作。
- **IValue data model**
  - EN: Moves values through generic boxed runtime containers used by TorchScript.
  - CN: 通过 TorchScript 使用的通用装箱运行时容器传递数值。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/api/module.h`, `torch/csrc/jit/ir/ir.h`, `torch/csrc/jit/serialization/export_bytecode.h`, `torch/csrc/jit/serialization/flatbuffer_serializer.h`, `torch/csrc/jit/serialization/python_print.h`, `torch/csrc/jit/serialization/storage_context.h`, `torch/csrc/jit/serialization/type_name_uniquer.h`, `torch/csrc/onnx/onnx.h`
- **Standard library / 标准库**: `caffe2/serialize/inline_container.h`, `ostream`
- **Primary symbols in this file / 本文件核心符号**: `ModelProto`, `TORCH_API`, `serialize_model_proto_to_string`, `check_onnx_proto`, `ScriptModuleSerializer`, `writeFiles`, `serialize`, `serialize_unified_format`

# export.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/serialization/export.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Serializes IValues, module state, or bytecode into portable binary/container representations.
- **Purpose (CN)**: 把 IValue、模块状态或字节码序列化为可移植的二进制/容器表示。
## Line-by-Line Analysis / 逐行分析

### Lines 1-27
```cpp
#include <torch/csrc/jit/serialization/export.h>

#include <ATen/ATen.h>
#include <ATen/Utils.h>
#include <ATen/core/functional.h>
#include <c10/macros/Macros.h>
#include <c10/util/Exception.h>
#include <c10/util/accumulate.h>
#include <c10/util/irange.h>
#include <torch/csrc/autograd/symbolic.h>
#include <torch/csrc/jit/jit_log.h>
#include <torch/csrc/jit/passes/dead_code_elimination.h>
#include <torch/csrc/jit/passes/inliner.h>
#include <torch/csrc/jit/runtime/instruction.h>
#include <torch/csrc/jit/serialization/import_export_constants.h>
#include <torch/csrc/jit/serialization/import_export_functions.h>
#include <torch/csrc/jit/serialization/import_export_helpers.h>
#include <torch/csrc/jit/serialization/onnx.h>
#include <torch/csrc/jit/serialization/pickler.h>
#include <torch/csrc/onnx/back_compat.h>
#include <torch/csrc/onnx/onnx.h>
#include <torch/version.h>

#include <onnx/checker.h>
#include <onnx/onnx_pb.h>
#include <onnx/proto_utils.h>
#include <onnx/shape_inference/implementation.h>
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/serialization/export.h, torch/csrc/autograd/symbolic.h, torch/csrc/jit/jit_log.h, and 11 more; ATen/c10 facilities such as ATen/ATen.h, ATen/Utils.h, ATen/core/functional.h, and 4 more; third-party components such as onnx/checker.h, onnx/onnx_pb.h, onnx/proto_utils.h, and 1 more.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/serialization/export.h、torch/csrc/autograd/symbolic.h、torch/csrc/jit/jit_log.h 等共 14 项；ATen/c10 基础设施，如 ATen/ATen.h、ATen/Utils.h、ATen/core/functional.h 等共 7 项；第三方组件，如 onnx/checker.h、onnx/onnx_pb.h、onnx/proto_utils.h 等共 4 项。

### Lines 29-61
```cpp
#include <memory>
#include <optional>
#include <regex>
#include <set>
#include <sstream>
#include <string>
#include <utility>
#include <vector>

namespace torch::jit {

static std::string get_little_endian_data(const at::Tensor& t) {
#if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
  return std::string(
      static_cast<char*>(t.data_ptr()), t.element_size() * t.numel());
#elif __BYTE_ORDER__ == __ORDER_BIG_ENDIAN__
  const size_t element_size = t.element_size();
  const size_t num_elements = t.numel();

  std::vector<char> data_copy{
      static_cast<char*>(t.data_ptr()),
      static_cast<char*>(t.data_ptr()) + element_size * num_elements};

  for (size_t i = 0; i < num_elements; ++i) {
    char* start_byte = data_copy.data() + i * element_size;
    char* end_byte = start_byte + element_size - 1;
    /* keep swapping */
    for (size_t count = 0; count < element_size / 2; ++count) {
      std::swap(*start_byte, *end_byte);
      ++start_byte;
      --end_byte;
    }
  }
```
- **EN**: This block assembles the compilation dependencies, pulling in standard-library headers such as memory, optional, regex, and 5 more. The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem. This chunk defines `get_little_endian_data`, which implements a focused step in loading or storing scripted programs. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段组织编译依赖，引入了标准库头文件，如 memory、optional、regex 等共 8 项。 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。 这一段定义了 `get_little_endian_data`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 63-88
```cpp
  return std::string(data_copy.data(), element_size * num_elements);
#else
#error Unexpected or undefined __BYTE_ORDER__
#endif
}

void writeArchiveAndTensors(
    const std::string& archive_name,
    const char* data,
    size_t size,
    const std::vector<at::Tensor>& tensors,
    caffe2::serialize::PyTorchStreamWriter& out) {
  std::string prefix = archive_name + "/";
  size_t i = 0;
  for (const auto& td : tensors) {
    WriteableTensorData writable_td = getWriteableTensorData(td);
    std::string fname = prefix + std::to_string(i++);
    out.writeRecord(fname, writable_td.data(), writable_td.sizeInBytes());
  }
  std::string fname = archive_name + ".pkl";
  out.writeRecord(fname, data, size);
}

namespace {
namespace onnx_torch = ::torch::onnx;
namespace onnx = ::ONNX_NAMESPACE;
```
- **EN**: The namespace declarations place the code inside onnx_torch, onnx, matching the surrounding JIT subsystem. This chunk defines `writeArchiveAndTensors`, which implements a focused step in loading or storing scripted programs. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 命名空间声明把代码放入 onnx_torch、onnx 中，与周边 JIT 子系统保持一致。 这一段定义了 `writeArchiveAndTensors`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 90-124
```cpp
constexpr int kInvalidOpsetVersion = -1;
constexpr int kMainOpsetVersion = 23;
// Based on OP_SET_ID_VERSION_MAP in
// https://github.com/onnx/onnx/blob/master/onnx/helper.py.
constexpr static std::array<int64_t, kMainOpsetVersion + 1>
    kOpsetVersionToIRVersion = {
        kInvalidOpsetVersion,
        3, // opset 1
        kInvalidOpsetVersion,
        kInvalidOpsetVersion,
        kInvalidOpsetVersion,
        3, // opset 5
        3, // opset 6
        3, // opset 7
        3, // opset 8
        4, // opset 9
        5, // opset 10
        6, // opset 11
        7, // opset 12
        7, // opset 13
        7, // opset 14
        8, // opset 15
        8, // opset 16
        8, // opset 17
        8, // opset 18
        9, // opset 19
        9, // opset 20
        10, // opset 21
        10, // opset 22
        11, // opset 23
};

std::string getNodeStackTraceString(const Node* n) {
  return n->sourceRange().str();
}
```
- **EN**: This chunk defines `getNodeStackTraceString`, which implements a focused step in loading or storing scripted programs. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getNodeStackTraceString`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 126-161
```cpp
void validateBlock(
    Block* b,
    onnx_torch::OperatorExportTypes operator_export_type) {
  for (auto node : b->nodes()) {
    for (Block* sub_block : node->blocks()) {
      validateBlock(sub_block, operator_export_type);
    }
    // Macro'ed so we get a marginally better line number on failed export
#define FAIL_EXPORT(name)                          \
  throw std::runtime_error(                        \
      std::string("ONNX export failed: ") + name + \
      "\n\nGraph we tried to export:\n" + b->owningGraph()->toString());
    // Special error messages for certain types of operators
    if (node->kind() == prim::PythonOp) {
      if (operator_export_type !=
          onnx_torch::OperatorExportTypes::ONNX_FALLTHROUGH) {
        auto py_node = static_cast<PythonOp*>(node);
        FAIL_EXPORT(
            "Couldn't export Python operator " + py_node->name() +
            "\n\nDefined at:\n" + getNodeStackTraceString(node))
      }
    } else {
      if (node->kind() == prim::PackPadded || node->kind() == prim::PadPacked) {
        if (operator_export_type !=
            onnx_torch::OperatorExportTypes::ONNX_FALLTHROUGH) {
          FAIL_EXPORT(
              "Cannot export individual pack_padded_sequence or pad_packed_sequence; these operations must occur in pairs.\n\nUsage of this operation occurred at:\n" +
              getNodeStackTraceString(node));
        }
      }
      bool is_aten_enabled = operator_export_type ==
              onnx_torch::OperatorExportTypes::ONNX_ATEN_FALLBACK ||
          operator_export_type == onnx_torch::OperatorExportTypes::ONNX_ATEN ||
          operator_export_type ==
              onnx_torch::OperatorExportTypes::ONNX_FALLTHROUGH;
      if (node->kind().is_aten() && !is_aten_enabled && !node->mustBeNone()) {
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 162-189
```cpp
        FAIL_EXPORT(
            "Couldn't export operator " + node->kind().toDisplayString() +
            "\n\nDefined at:\n" + getNodeStackTraceString(node));
      }
    }
#undef FAIL_EXPORT
  }
}

void validateGraph(
    const std::shared_ptr<Graph>& graph,
    onnx_torch::OperatorExportTypes operator_export_type) {
  validateBlock(graph->block(), operator_export_type);
}

std::string GetFileRootPath(const std::string& rootPath) {
  std::string rootPath_ = rootPath;
  // First, making slash consistent.
  std::replace(rootPath_.begin(), rootPath_.end(), '\\', '/');
  // Second, remove trailing slashes, if any
  std::regex trailer("/+$");
  std::string root = std::regex_replace(rootPath_, trailer, std::string());
  std::string folder = root.substr(0, root.find_last_of('/'));
  if (folder == rootPath_) { // If no root folder specified, select cwd.
    return std::string(".");
  }
  return folder;
}
```
- **EN**: This chunk defines `trailer`, which implements a focused step in loading or storing scripted programs. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `trailer`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 191-222
```cpp
std::string GetExternalFileName(
    const std::optional<std::string>& external_ref) {
  auto tensorName = external_ref.value();
  const std::string illegalChars = "\\/:?\"<>|";
  for (char& i : tensorName) {
    if (illegalChars.find(i) != std::string::npos) {
      i = '_';
    }
  }
  return tensorName;
}

void CloseFile(FILE* fp) {
  fclose(fp);
}

void CreateExternalFile(
    const at::Tensor& tensor,
    const std::string& tensorName,
    const std::string& onnx_file_path) {
  auto folder = GetFileRootPath(onnx_file_path);
  std::string fullFilePath = folder + "/" + tensorName;
  std::unique_ptr<FILE, decltype(&CloseFile)> fp(
      fopen(fullFilePath.c_str(), "wb"), &CloseFile);
  if (fp == nullptr) {
    throw std::runtime_error(
        std::string("ONNX export failed. Could not open file or directory: ") +
        fullFilePath);
  }
  std::string s = get_little_endian_data(tensor);
  fwrite(s.c_str(), tensor.element_size(), tensor.numel(), fp.get());
} // fclose() called here through CloseFile(), if FILE* is not a null pointer.
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 224-257
```cpp
class GraphEncoder {
 public:
  GraphEncoder(
      const std::shared_ptr<Graph>& graph,
      int64_t onnx_opset_version,
      onnx_torch::OperatorExportTypes operator_export_type,
      const std::map<std::string, at::Tensor>& initializers,
      const std::unordered_map<
          std::string,
          std::unordered_map<int64_t, std::string>>& dynamic_axes,
      bool defer_weight_export,
      bool strip_doc,
      bool keep_initializers_as_inputs,
      const std::map<std::string, int>& custom_opsets,
      bool add_node_names,
      bool use_external_data_format,
      const std::string& onnx_file_path,
      NodeAttrNameMap node_attr_to_name = {});

  std::shared_ptr<onnx::ModelProto> get_model_proto() {
    return model_proto_;
  }

  SymbolDimMap get_symbol_dim_param_map() {
    return symbol_dim_map_;
  }

  RawDataExportMap get_raw_data_export_map() {
    return raw_data_export_map_;
  }

  bool get_use_external_data_format() {
    return use_external_data_format_;
  }
```
- **EN**: It introduces or extends GraphEncoder, which define the primary data structures or interfaces for this portion of the file. This chunk defines `get_use_external_data_format`, which implements a focused step in loading or storing scripted programs. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 它引入或扩展了 GraphEncoder，这些类型定义了本段涉及的主要数据结构或接口。 这一段定义了 `get_use_external_data_format`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 259-283
```cpp
  NodeNameMap get_onnx_node_names() {
    return onnx_node_name_map_;
  }

 private:
  // Using std::map instead of std::unordered_map for initializers
  // in EncodeGraph constructor so that the order in which initializers
  // get written to the ONNX graph is always the deterministic and
  // predictable. While this is not a ONNX requirement, it is needed
  // for testing purposes in tests that use _export_to_pretty_string()
  // for validating ONNX graphs.
  void EncodeGraph(
      onnx::GraphProto* graph_proto,
      const std::shared_ptr<Graph>& graph,
      const std::map<std::string, at::Tensor>& initializers =
          std::map<std::string, at::Tensor>(),
      const std::
          unordered_map<std::string, std::unordered_map<int64_t, std::string>>&
              dynamic_axes = std::unordered_map<
                  std::string,
                  std::unordered_map<int64_t, std::string>>(),
      bool keep_initializers_as_inputs = true,
      bool add_node_names = true,
      bool use_external_data_format = false,
      const std::string& onnx_file_path = std::string());
```
- **EN**: This chunk defines `get_onnx_node_names`, which implements a focused step in loading or storing scripted programs. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `get_onnx_node_names`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 285-315
```cpp
  void EncodeBlock(
      onnx::GraphProto* graph_proto,
      const Block* block,
      const std::map<std::string, at::Tensor>& initializers =
          std::map<std::string, at::Tensor>(),
      const std::
          unordered_map<std::string, std::unordered_map<int64_t, std::string>>&
              dynamic_axes = std::unordered_map<
                  std::string,
                  std::unordered_map<int64_t, std::string>>(),
      bool keep_initializers_as_inputs = true,
      bool add_node_names = true,
      bool use_external_data_format = false,
      const std::string& onnx_file_path = std::string());

  void AddInitializersIntoGraphProto(
      onnx::GraphProto* graph_proto,
      const Block* block,
      const std::map<std::string, at::Tensor>& initializers =
          std::map<std::string, at::Tensor>(),
      bool use_external_data_format = false,
      const std::string& onnx_file_path = std::string());

  unsigned long long int GetGraphProtoSize(
      onnx::GraphProto* graph_proto,
      const std::shared_ptr<Graph>& graph,
      bool add_node_names,
      bool use_external_data_format,
      const std::string& onnx_file_path,
      const std::map<std::string, at::Tensor>& initializers =
          std::map<std::string, at::Tensor>());
```
- **EN**: This chunk continues `get_onnx_node_names` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_onnx_node_names`，进一步展开其内部控制流或数据流转。

### Lines 317-348
```cpp
  void EncodeNode(
      onnx::GraphProto* graph_proto,
      onnx::NodeProto* node_proto,
      const Node* node,
      bool add_node_names = true,
      bool use_external_data_format = false,
      const std::string& onnx_file_path = std::string());

  void EncodeTypeProto(
      onnx::TypeProto* type_proto,
      const TypePtr& node_type,
      const std::string& name);

  void EncodeLocalFunctionOpsetImport(
      onnx::FunctionProto* func_proto,
      const Node* n,
      std::unordered_set<std::string>& custom_domains);

  void EncodeLocalFunction(
      onnx::GraphProto* graph_proto,
      onnx::FunctionProto* func_proto,
      const Node* n,
      bool add_node_names = true,
      bool use_external_data_format = false,
      const std::string& onnx_file_path = std::string());

  void EncodeTensor(
      onnx::TensorProto* tensor_proto,
      const at::Tensor& tensor,
      const std::optional<std::string>& external_ref = {},
      const bool use_external_data_format = false,
      const std::string& onnx_file_path = std::string());
```
- **EN**: This chunk defines `EncodeLocalFunctionOpsetImport`, which imports stored definitions into live runtime or compilation-unit objects.
- **CN**: 这一段定义了 `EncodeLocalFunctionOpsetImport`，其作用是把已存储定义导入为可用的运行时或编译单元对象。

### Lines 350-383
```cpp
  void EncodeIntermediateValueInfo(
      onnx::GraphProto* graph_proto,
      const Value* n);

  void EncodeValueInfo(
      onnx::GraphProto* graph_proto,
      onnx::ValueInfoProto* v,
      const Value* n,
      const std::
          unordered_map<std::string, std::unordered_map<int64_t, std::string>>&
              dynamic_axes = std::unordered_map<
                  std::string,
                  std::unordered_map<int64_t, std::string>>());

  void EncodeValueInfoType(
      onnx::TypeProto* onnx_type,
      const TypePtr& node_type,
      const Value* n,
      const std::unordered_map<
          std::string,
          std::unordered_map<int64_t, std::string>>& dynamic_axes);

  void AddAttribute(
      onnx::NodeProto* node_proto,
      const jit::Symbol name,
      const std::string& ref_attr_name,
      const AttributeKind attr_kind);

  void AddAttribute(
      onnx::NodeProto* node_proto,
      const jit::Node* node,
      const jit::Symbol name,
      const bool use_external_data_format = false,
      const std::string& onnx_file_path = std::string());
```
- **EN**: This chunk declares `AddAttribute`, which implements a focused step in loading or storing scripted programs.
- **CN**: 这一段声明了 `AddAttribute`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。

### Lines 385-419
```cpp
  void AddAttribute(onnx::FunctionProto* func_proto, const std::string& name);

  void TensorTypeToONNXType(
      const TensorTypePtr& tensor_type,
      const std::string& dim_name_prefix,
      const std::string& name,
      const std::unordered_map<
          std::string,
          std::unordered_map<int64_t, std::string>>& dynamic_axes,
      onnx::TypeProto_Tensor* onnx_tensor_type,
      bool assign_dim_param = true);

  SymbolDimMap symbol_dim_map_;
  std::shared_ptr<onnx::ModelProto> model_proto_;
  size_t num_blocks_{0};
  size_t num_op_nodes_{0};
  size_t num_external_data_{0};
  onnx_torch::OperatorExportTypes operator_export_type_;
  bool strip_doc_;
  std::set<std::string> domains_;
  RawDataExportMap raw_data_export_map_;
  bool defer_weight_export_;
  bool use_external_data_format_;
  int64_t onnx_opset_version_;
  std::map<std::string, int> custom_opsets_;
  std::shared_ptr<Graph> graph_;
  NodeAttrNameMap node_attr_to_name_;
  NodeNameMap onnx_node_name_map_;
  // For large models, the parameters can be stored in separate binary files.
  // This parameter sets a threshold on the number of elements in the parameter
  // tensor, beyond which the parameter is stored in a separate file (if
  // use_external_data_format_ is True). This threshold is in place
  // so as not to create too many external files.
  static constexpr size_t ParamSizeThresholdForExternalStorage = 1024;
};
```
- **EN**: This chunk defines `AddAttribute`, which implements a focused step in loading or storing scripted programs.
- **CN**: 这一段定义了 `AddAttribute`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。

### Lines 421-456
```cpp
static onnx::TensorProto_DataType ATenTypeToOnnxType(at::ScalarType at_type) {
  switch (at_type) {
    case at::kDouble:
      return onnx::TensorProto_DataType_DOUBLE;
    case at::kFloat:
      return onnx::TensorProto_DataType_FLOAT;
    case at::kHalf:
      return onnx::TensorProto_DataType_FLOAT16;
    case at::kByte:
      return onnx::TensorProto_DataType_UINT8;
    case at::kChar:
      return onnx::TensorProto_DataType_INT8;
    case at::kShort:
      return onnx::TensorProto_DataType_INT16;
    case at::kInt:
      return onnx::TensorProto_DataType_INT32;
    case at::kLong:
      return onnx::TensorProto_DataType_INT64;
    case at::kBool:
      return onnx::TensorProto_DataType_BOOL;
    case at::kQInt8:
      return onnx::TensorProto_DataType_INT8;
    case at::kQUInt8:
      return onnx::TensorProto_DataType_UINT8;
    case at::kQInt32:
      return onnx::TensorProto_DataType_INT32;
    case at::kBFloat16:
      return onnx::TensorProto_DataType_BFLOAT16;
    case at::kFloat8_e4m3fn:
      return onnx_torch::TensorProto_DataType_FLOAT8E4M3FN;
    case at::kFloat8_e5m2:
      return onnx_torch::TensorProto_DataType_FLOAT8E5M2;
    case at::kFloat8_e4m3fnuz:
      return onnx_torch::TensorProto_DataType_FLOAT8E4M3FNUZ;
    case at::kFloat8_e5m2fnuz:
      return onnx_torch::TensorProto_DataType_FLOAT8E5M2FNUZ;
```
- **EN**: This chunk defines `ATenTypeToOnnxType`, which implements a focused step in loading or storing scripted programs. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `ATenTypeToOnnxType`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 457-492
```cpp
    default:
      TORCH_CHECK(
          false,
          "ScalarType ",
          toString(at_type),
          " is an unexpected tensor scalar type");
  }
}

static onnx::AttributeProto_AttributeType ATenAttributeKindToOnnxAttributeType(
    AttributeKind at_kind,
    const jit::Symbol name) {
  switch (at_kind) {
    case AttributeKind::f:
      return onnx::AttributeProto_AttributeType_FLOAT;
    case AttributeKind::fs:
      return onnx::AttributeProto_AttributeType_FLOATS;
    case AttributeKind::i:
      return onnx::AttributeProto_AttributeType_INT;
    case AttributeKind::is:
      return onnx::AttributeProto_AttributeType_INTS;
    case AttributeKind::s:
      return onnx::AttributeProto_AttributeType_STRING;
    case AttributeKind::ss:
      return onnx::AttributeProto_AttributeType_STRINGS;
    case AttributeKind::t:
      return onnx::AttributeProto_AttributeType_TENSOR;
    case AttributeKind::ts:
      return onnx::AttributeProto_AttributeType_TENSORS;
    case AttributeKind::ty:
      return onnx::AttributeProto_AttributeType_TYPE_PROTO;
    case AttributeKind::tys:
      return onnx::AttributeProto_AttributeType_TYPE_PROTOS;
    case AttributeKind::g:
      return onnx::AttributeProto_AttributeType_GRAPH;
    case AttributeKind::gs:
```
- **EN**: This chunk defines `ATenAttributeKindToOnnxAttributeType`, which implements a focused step in loading or storing scripted programs. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `ATenAttributeKindToOnnxAttributeType`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 493-518
```cpp
      return onnx::AttributeProto_AttributeType_GRAPHS;
    default:
      std::ostringstream err_msg;
      err_msg << "attribute \"" << name.toDisplayString()
              << "\" has unexpected kind: " << toString(at_kind);
      throw std::runtime_error(err_msg.str());
  }
}

GraphEncoder::GraphEncoder(
    const std::shared_ptr<Graph>& graph,
    int64_t onnx_opset_version,
    onnx_torch::OperatorExportTypes operator_export_type,
    const std::map<std::string, at::Tensor>& initializers,
    const std::unordered_map<
        std::string,
        std::unordered_map<int64_t, std::string>>& dynamic_axes,
    bool defer_weight_export,
    bool strip_doc,
    bool keep_initializers_as_inputs,
    const std::map<std::string, int>& custom_opsets,
    bool add_node_names,
    bool use_external_data_format,
    const std::string& onnx_file_path,
    NodeAttrNameMap node_attr_to_name)
    : model_proto_(std::make_shared<onnx::ModelProto>()),
```
- **EN**: This chunk declares `runtime_error`, which executes prepared logic against runtime values or IR state. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段声明了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 520-555
```cpp
      operator_export_type_(operator_export_type),
      strip_doc_(strip_doc),
      defer_weight_export_(defer_weight_export),
      use_external_data_format_(use_external_data_format),
      onnx_opset_version_(onnx_opset_version),
      custom_opsets_(custom_opsets),
      graph_(graph),
      node_attr_to_name_(std::move(node_attr_to_name)) {
  model_proto_->set_producer_name("pytorch");
  TORCH_CHECK(
      onnx_opset_version > 0 &&
          static_cast<size_t>(onnx_opset_version) <
              kOpsetVersionToIRVersion.size() &&
          kOpsetVersionToIRVersion[onnx_opset_version] != kInvalidOpsetVersion,
      "Unsupported onnx_opset_version: ",
      onnx_opset_version);

  model_proto_->set_ir_version(kOpsetVersionToIRVersion[onnx_opset_version]);
  model_proto_->set_producer_version(TORCH_VERSION);
  validateGraph(graph, operator_export_type);

  // If graph proto size exceed maximum protobuf size of 2GB, set
  // use_external_data_format to true.
  if (!use_external_data_format &&
      GetGraphProtoSize(
          model_proto_->mutable_graph(),
          graph,
          add_node_names,
          use_external_data_format,
          onnx_file_path,
          initializers) > INT_MAX) {
    GRAPH_DEBUG(
        "Exporting model exceed maximum protobuf size of 2GB. Storing model parameters in external data files");
    use_external_data_format = true;
    // use_external_data_format_ is one of graph_encoder private variable set
    // for return `use_external_data_format` value.
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 556-579
```cpp
    use_external_data_format_ = use_external_data_format;
  }

  if (use_external_data_format) {
    TORCH_CHECK(
        !onnx_file_path.empty(),
        "The serialized model is larger than the 2GiB limit imposed by the protobuf library. ",
        "Therefore the output file must be a file path, so that the ONNX external data can ",
        "be written to the same directory. Please specify the output file name.");
  }

  auto* imp = model_proto_->add_opset_import();
  // This is the version of ONNX operator set we are targeting
  imp->set_version(onnx_opset_version);

  EncodeGraph(
      model_proto_->mutable_graph(),
      graph,
      initializers,
      dynamic_axes,
      keep_initializers_as_inputs,
      add_node_names,
      use_external_data_format,
      onnx_file_path);
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 581-616
```cpp
  for (const std::string& domain : domains_) {
    auto* opset = model_proto_->add_opset_import();
    opset->set_domain(domain);
    //  Check if domain version is registered. If not, set to version 1
    auto it = custom_opsets.find(domain);
    if (it == custom_opsets.end())
      opset->set_version(1);
    else {
      opset->set_version(it->second);
    }
  }

  for (auto const& custom_opset : custom_opsets) {
    if (!std::count(domains_.begin(), domains_.end(), custom_opset.first)) {
      TORCH_WARN(
          "Custom opset domain: '",
          custom_opset.first,
          "' provided is not used in the model. ",
          "Please verify custom opset domain names.");
    }
  }
}

// NOLINTBEGIN(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
void GraphEncoder::TensorTypeToONNXType(
    const TensorTypePtr& tensor_type,
    const std::string& dim_name_prefix,
    const std::string& name,
    const std::unordered_map<
        std::string,
        std::unordered_map<int64_t, std::string>>& dynamic_axes,
    onnx::TypeProto_Tensor* onnx_tensor_type,
    bool assign_dim_param) {
  if (tensor_type->dim()) {
    onnx::TensorShapeProto* shape = onnx_tensor_type->mutable_shape();
    auto sizes = tensor_type->symbolic_sizes().sizes().value();
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 617-641
```cpp
    for (const auto i : c10::irange(sizes.size())) {
      shape->add_dim();
      if ((dynamic_axes.find(name) != dynamic_axes.end()) &&
          (dynamic_axes.at(name).find(i) != dynamic_axes.at(name).end())) {
        shape->mutable_dim(i)->set_dim_param(dynamic_axes.at(name).at(i));
        if (!sizes[i].is_static()) {
          symbol_dim_map_[sizes[i]] = dynamic_axes.at(name).at(i);
        }
      } else if (sizes[i].is_static()) {
        shape->mutable_dim(i)->set_dim_value(sizes[i].static_size());
      } else if (assign_dim_param) {
        if (symbol_dim_map_.find(sizes[i]) == symbol_dim_map_.end()) {
          symbol_dim_map_[sizes[i]] =
              dim_name_prefix + name + "_dim_" + std::to_string(i);
        }
        shape->mutable_dim(i)->set_dim_param(symbol_dim_map_[sizes[i]]);
      }
    }
  }
  if (tensor_type->scalarType()) {
    onnx_tensor_type->set_elem_type(
        ATenTypeToOnnxType(tensor_type->scalarType().value()));
  }
}
// NOLINTEND(bugprone-narrowing-conversions,cppcoreguidelines-narrowing-conversions)
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 643-678
```cpp
void GraphEncoder::EncodeValueInfoType(
    onnx::TypeProto* onnx_type,
    const TypePtr& node_type,
    const Value* n,
    const std::unordered_map<
        std::string,
        std::unordered_map<int64_t, std::string>>& dynamic_axes) {
  std::string dim_name_prefix;
  if (n->node()->kind() != prim::Param) {
    dim_name_prefix = n->node()->kind().toUnqualString();
  }
  if (TensorTypePtr tensor_type = node_type->cast<TensorType>()) {
    if (tensor_type->dim() || tensor_type->scalarType()) {
      // Encode type if either shape or dtype exists.
      onnx::TypeProto_Tensor* onnx_tensor_type =
          onnx_type->mutable_tensor_type();
      // Do not assign dim_param for sequence tensor type.
      // Sequence of tensors could differ in dimension size.
      // Use a dimension with neither dim_value nor dim_param set
      // to denote an unknown dimension.
      // Create and assign dim_param for normal tensor type.
      auto is_sequence_tensor = static_cast<bool>(n->type()->cast<ListType>());
      TensorTypeToONNXType(
          tensor_type,
          dim_name_prefix,
          n->debugName(),
          dynamic_axes,
          onnx_tensor_type,
          !is_sequence_tensor);
    }
  } else if (BoolTypePtr bool_type = node_type->cast<BoolType>()) {
    onnx::TypeProto_Tensor* onnx_tensor_type = onnx_type->mutable_tensor_type();
    onnx_tensor_type->set_elem_type(ATenTypeToOnnxType(at::kBool));
  } else if (IntTypePtr int_type = node_type->cast<IntType>()) {
    onnx::TypeProto_Tensor* onnx_tensor_type = onnx_type->mutable_tensor_type();
    onnx_tensor_type->set_elem_type(ATenTypeToOnnxType(at::kLong));
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 679-714
```cpp
  } else if (FloatTypePtr float_type = node_type->cast<FloatType>()) {
    onnx::TypeProto_Tensor* onnx_tensor_type = onnx_type->mutable_tensor_type();
    onnx_tensor_type->set_elem_type(ATenTypeToOnnxType(at::kFloat));
  } else if (ListTypePtr list_type = node_type->cast<ListType>()) {
    auto list_elem_type = list_type->getElementType();
    onnx::TypeProto_Sequence* sequence_type =
        onnx_type->mutable_sequence_type();
    onnx::TypeProto* onnx_tensor_type = sequence_type->mutable_elem_type();
    EncodeValueInfoType(onnx_tensor_type, list_elem_type, n, dynamic_axes);
  } else if (OptionalTypePtr optional_type = node_type->cast<OptionalType>()) {
    auto elem_type = optional_type->getElementType();
    if (TensorTypePtr tensor_type = elem_type->cast<TensorType>()) {
      onnx::TypeProto_Optional* onnx_optional_type =
          onnx_type->mutable_optional_type();
      onnx::TypeProto_Tensor* onnx_tensor_type =
          onnx_optional_type->mutable_elem_type()->mutable_tensor_type();
      TensorTypeToONNXType(
          tensor_type,
          dim_name_prefix,
          n->debugName(),
          dynamic_axes,
          onnx_tensor_type);
    } else if (ListTypePtr inner_node_type = elem_type->cast<ListType>()) {
      auto list_elem_type = inner_node_type->getElementType();
      if (TensorTypePtr tensor_type = list_elem_type->cast<TensorType>()) {
        onnx::TypeProto_Optional* onnx_optional_type =
            onnx_type->mutable_optional_type();
        onnx::TypeProto_Sequence* onnx_optional_sequence_type =
            onnx_optional_type->mutable_elem_type()->mutable_sequence_type();
        onnx::TypeProto_Tensor* onnx_tensor_type =
            onnx_optional_sequence_type->mutable_elem_type()
                ->mutable_tensor_type();
        TensorTypeToONNXType(
            tensor_type,
            dim_name_prefix,
            n->debugName(),
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 715-750
```cpp
            dynamic_axes,
            onnx_tensor_type);
      }
    }
  }
}

void GraphEncoder::EncodeValueInfo(
    onnx::GraphProto* graph_proto,
    onnx::ValueInfoProto* v,
    const Value* n,
    const std::unordered_map<
        std::string,
        std::unordered_map<int64_t, std::string>>& dynamic_axes) {
  std::string name = n->debugName();
  v->set_name(name);
  EncodeValueInfoType(v->mutable_type(), n->type(), n, dynamic_axes);
}

void GraphEncoder::EncodeGraph(
    onnx::GraphProto* graph_proto,
    const std::shared_ptr<Graph>& graph,
    const std::map<std::string, at::Tensor>& initializers,
    const std::unordered_map<
        std::string,
        std::unordered_map<int64_t, std::string>>& dynamic_axes,
    bool keep_initializers_as_inputs,
    bool add_node_names,
    bool use_external_data_format,
    const std::string& onnx_file_path) {
  EncodeBlock(
      graph_proto,
      graph->block(),
      initializers,
      dynamic_axes,
      keep_initializers_as_inputs,
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。

### Lines 751-779
```cpp
      add_node_names,
      use_external_data_format,
      onnx_file_path);
}

void GraphEncoder::EncodeBlock(
    onnx::GraphProto* graph_proto,
    const Block* block,
    const std::map<std::string, at::Tensor>& initializers,
    const std::unordered_map<
        std::string,
        std::unordered_map<int64_t, std::string>>& dynamic_axes,
    bool keep_initializers_as_inputs,
    bool add_node_names,
    bool use_external_data_format,
    const std::string& onnx_file_path) {
  TORCH_INTERNAL_ASSERT(graph_proto != nullptr);
  if (nullptr == block->owningNode()) {
    // Top level main graph.
    graph_proto->set_name("main_graph");
  } else {
    // TODO: Set more meaningful name for sub-graphs.
    std::string block_name = "sub_graph";
    if (num_blocks_) {
      block_name += std::to_string(num_blocks_);
    }
    num_blocks_++;
    graph_proto->set_name(block_name);
  }
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 781-816
```cpp
  // Since ONNX IR VERSION 4, initializers do not have to
  // be a subset of graph inputs. We use keep_initializers_as_inputs
  // argument to determine whether to add initializers
  // as inputs or not. If keep_initializers_as_inputs=false,
  // we only add non-parameter inputs as inputs to ONNX graph, and
  // not the initializers (parameters). If keep_initializers_as_inputs
  // =true, we add initializers as inputs too. Setting
  // keep_initializers_as_inputs=false allows better
  // optimizations, such as constant-folding, on ONNX graphs
  // by backends/optimizers.
  if (keep_initializers_as_inputs) {
    for (auto input : block->inputs()) {
      onnx::ValueInfoProto* v = graph_proto->add_input();
      EncodeValueInfo(graph_proto, v, input, dynamic_axes);
    }
  } else {
    for (auto input : block->inputs()) {
      auto it = initializers.find(input->debugName());
      if (it == initializers.end()) {
        onnx::ValueInfoProto* v = graph_proto->add_input();
        EncodeValueInfo(graph_proto, v, input, dynamic_axes);
      }
    }
  }
  for (auto output : block->outputs()) {
    onnx::ValueInfoProto* v = graph_proto->add_output();
    EncodeValueInfo(graph_proto, v, output, dynamic_axes);
  }
  for (auto node : block->nodes()) {
    if (node->mustBeNone()) {
      // None nodes are used to implement optional inputs. One
      // way to "not provide" an optional input is to create an
      // Undefined node, and pass its output as that input.
      continue;
    }
    if (node->kind() == ::c10::Symbol::onnx("LocalFunctionDef")) {
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 817-842
```cpp
      auto* func_proto = model_proto_->add_functions();
      EncodeLocalFunction(
          graph_proto,
          func_proto,
          node,
          add_node_names,
          use_external_data_format,
          onnx_file_path);
      continue;
    }
    auto* n_proto = graph_proto->add_node();
    EncodeNode(
        graph_proto,
        n_proto,
        node,
        add_node_names,
        use_external_data_format,
        onnx_file_path);
  }
  AddInitializersIntoGraphProto(
      graph_proto,
      block,
      initializers,
      use_external_data_format,
      onnx_file_path);
}
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。

### Lines 844-874
```cpp
void GraphEncoder::AddInitializersIntoGraphProto(
    onnx::GraphProto* graph_proto,
    const Block* block,
    const std::map<std::string, at::Tensor>& initializers,
    bool use_external_data_format,
    const std::string& onnx_file_path) {
  TORCH_INTERNAL_ASSERT(block->inputs().size() >= initializers.size());
  for (auto input : block->inputs()) {
    auto name_tensor_pair = initializers.find(input->debugName());
    if (name_tensor_pair == initializers.end()) {
      continue;
    }
    auto p = graph_proto->add_initializer();
    p->set_name(name_tensor_pair->first);
    EncodeTensor(
        p,
        name_tensor_pair->second,
        name_tensor_pair->first,
        use_external_data_format,
        onnx_file_path);
  }
}

unsigned long long int GraphEncoder::GetGraphProtoSize(
    onnx::GraphProto* graph_proto,
    const std::shared_ptr<Graph>& graph,
    bool add_node_names,
    bool use_external_data_format,
    const std::string& onnx_file_path,
    const std::map<std::string, at::Tensor>& initializers) {
  // Model size = sum(size(initializers)) + sum(size(onnx_constant_nodes))
```
- **EN**: This chunk defines `AddInitializersIntoGraphProto`, which implements a focused step in loading or storing scripted programs. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `AddInitializersIntoGraphProto`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 876-902
```cpp
  // Add up all Initializers
  onnx::GraphProto graph_proto_copy = onnx::GraphProto(*graph_proto);
  unsigned long long int size = graph_proto_copy.ByteSizeLong();
  for (auto input : graph->inputs()) {
    auto name_tensor_pair = initializers.find(input->debugName());
    if (name_tensor_pair == initializers.end()) {
      continue;
    }
    auto tensor_proto = graph_proto_copy.add_initializer();
    const at::Tensor& tensor = name_tensor_pair->second;
    for (auto d : tensor.sizes()) {
      tensor_proto->add_dims(d);
    }
    tensor_proto->set_data_type(ATenTypeToOnnxType(tensor.scalar_type()));

    // Don't actually copy the buffer into tensor_proto since that is expensive.
    // All we actually need is its size.
    size += tensor_proto->ByteSizeLong();
    size += tensor.element_size() * tensor.numel();
  }

  // Add up all onnx::Constant nodes that are Tensors
  for (const auto& node : graph->nodes()) {
    if (node->kind() == ::c10::onnx::Constant &&
        node->hasAttribute(attr::value) &&
        node->kindOf(attr::value) == AttributeKind::t) {
      at::Tensor tensor = node->t(attr::value);
```
- **EN**: This chunk continues `AddInitializersIntoGraphProto` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `AddInitializersIntoGraphProto`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 904-939
```cpp
      // Don't actually copy the buffer into n_proto since that is expensive.
      // All we actually need is its size.
      auto* n_proto = graph_proto_copy.add_node();
      EncodeNode(
          &graph_proto_copy,
          n_proto,
          node,
          add_node_names,
          use_external_data_format,
          onnx_file_path);

      // Calculate the size of the tensor in bytes
      size += n_proto->ByteSizeLong();
      size += tensor.element_size() * tensor.numel();
    }
  }
  return size;
}

void GraphEncoder::EncodeNode(
    onnx::GraphProto* graph_proto,
    onnx::NodeProto* node_proto,
    const Node* node,
    bool add_node_names,
    bool use_external_data_format,
    const std::string& onnx_file_path) {
  if (!strip_doc_) {
    node_proto->set_doc_string(node->sourceRange().str());
  }
  for (auto input : node->inputs()) {
    if (input->node()->mustBeNone()) {
      node_proto->add_input("");
    } else {
      node_proto->add_input(input->debugName());
    }
  }
```
- **EN**: This chunk continues `AddInitializersIntoGraphProto` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `AddInitializersIntoGraphProto`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 940-975
```cpp
  for (auto output : node->outputs()) {
    node_proto->add_output(output->debugName());
    EncodeIntermediateValueInfo(graph_proto, output);
  }
  if (!node->kind().is_onnx()) {
    std::string domain;
    if (node->kind().is_aten() || node->kind().is_caffe2()) {
      domain = node->kind().domainString();
    } else { //  Custom namespace and domain
      domain = node->kind().ns().toUnqualString();
    }
    // TODO: set correct domain for function proto.
    domains_.insert(domain);
    node_proto->set_domain(domain);
  }
  if (operator_export_type_ == onnx_torch::OperatorExportTypes::ONNX) {
    TORCH_INTERNAL_ASSERT(
        !node->kind().is_aten() && !node->kind().is_prim() &&
        !node->kind().is_attr());
  }
  node_proto->set_op_type(node->kind().toUnqualString());
  const auto node_name_attribute_symbol =
      Symbol::attr(::torch::onnx::kOnnxNodeNameAttribute);
  if (add_node_names) {
    std::string node_name =
        node_proto->op_type() + "_" + std::to_string(num_op_nodes_);
    if (node->hasAttribute(node_name_attribute_symbol)) {
      node_name = node->s(node_name_attribute_symbol);
    }
    node_proto->set_name(node_name);
    onnx_node_name_map_[node] = node_name;
    num_op_nodes_++;
  }
  auto attrs_it = node_attr_to_name_.find(node);
  for (auto attr_name : node->attributeNames()) {
    if (attr_name == node_name_attribute_symbol) {
```
- **EN**: This chunk continues `AddInitializersIntoGraphProto` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `AddInitializersIntoGraphProto`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 976-1008
```cpp
      // Skip the node name attribute.
      continue;
    }
    if (attrs_it != node_attr_to_name_.end()) {
      auto attr_it = attrs_it->second.find(attr_name.toUnqualString());
      if (attr_it != attrs_it->second.end()) {
        AddAttribute(
            node_proto, attr_name, attr_it->second, node->kindOf(attr_name));
        continue;
      }
    }
    AddAttribute(
        node_proto, node, attr_name, use_external_data_format, onnx_file_path);
  }
  if (node->kind() == ::c10::onnx::Loop) {
    TORCH_INTERNAL_ASSERT(node->blocks().size() == 1);

    auto body = node_proto->add_attribute();
    body->set_name("body");
    body->set_type(onnx::AttributeProto_AttributeType_GRAPH);
    auto g = body->mutable_g();
    EncodeBlock(
        g,
        node->blocks()[0],
        {},
        {},
        true,
        true,
        use_external_data_format,
        onnx_file_path);
  }
  if (node->kind() == ::c10::onnx::If) {
    TORCH_INTERNAL_ASSERT(node->blocks().size() == 2);
```
- **EN**: This chunk continues `AddInitializersIntoGraphProto` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `AddInitializersIntoGraphProto`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1010-1038
```cpp
    auto then_branch = node_proto->add_attribute();
    then_branch->set_name("then_branch");
    then_branch->set_type(onnx::AttributeProto_AttributeType_GRAPH);
    auto true_g = then_branch->mutable_g();
    EncodeBlock(
        true_g,
        node->blocks()[0],
        {},
        {},
        true,
        true,
        use_external_data_format,
        onnx_file_path);

    auto else_branch = node_proto->add_attribute();
    else_branch->set_name("else_branch");
    else_branch->set_type(onnx::AttributeProto_AttributeType_GRAPH);
    auto false_g = else_branch->mutable_g();
    EncodeBlock(
        false_g,
        node->blocks()[1],
        {},
        {},
        true,
        true,
        use_external_data_format,
        onnx_file_path);
  }
}
```
- **EN**: This chunk continues `AddInitializersIntoGraphProto` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `AddInitializersIntoGraphProto`，进一步展开其内部控制流或数据流转。

### Lines 1040-1074
```cpp
void GraphEncoder::AddAttribute(
    onnx::NodeProto* node_proto,
    const jit::Symbol name,
    const std::string& ref_attr_name,
    const AttributeKind attr_kind) {
  auto attr = node_proto->add_attribute();
  TORCH_INTERNAL_ASSERT(name.is_attr());
  attr->set_name(name.toUnqualString());
  attr->set_ref_attr_name(ref_attr_name);
  attr->set_type(ATenAttributeKindToOnnxAttributeType(attr_kind, name));
}

void GraphEncoder::AddAttribute(
    onnx::NodeProto* node_proto,
    const jit::Node* node,
    const jit::Symbol name,
    const bool use_external_data_format,
    const std::string& onnx_file_path) {
  auto createAttributeTensorName =
      [](const onnx::NodeProto* node_proto,
         onnx::TensorProto* tensor_proto,
         const jit::Symbol attr_name,
         size_t& num_external_data) -> std::string {
    if (tensor_proto->has_name()) {
      return tensor_proto->name();
    }
    if (!node_proto->has_name()) {
      auto name = node_proto->op_type() + "_" + attr_name.toDisplayString() +
          "_" + std::to_string(num_external_data);
      num_external_data++;
      return name;
    } else {
      return node_proto->name() + "_" + attr_name.toDisplayString();
    }
  };
```
- **EN**: This chunk defines `AddAttribute`, which implements a focused step in loading or storing scripted programs. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `AddAttribute`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1076-1111
```cpp
  auto attr = node_proto->add_attribute();
  TORCH_INTERNAL_ASSERT(name.is_attr());
  attr->set_name(name.toUnqualString());
  attr->set_type(
      ATenAttributeKindToOnnxAttributeType(node->kindOf(name), name));
  switch (node->kindOf(name)) {
    case AttributeKind::f:
      attr->set_f(static_cast<float>(node->f(name)));
      break;
    case AttributeKind::fs:
      for (auto& v : node->fs(name))
        attr->add_floats(static_cast<float>(v));
      break;
    case AttributeKind::i:
      attr->set_i(node->i(name));
      break;
    case AttributeKind::is:
      for (auto& v : node->is(name))
        attr->add_ints(v);
      break;
    case AttributeKind::s:
      attr->set_s(node->s(name));
      break;
    case AttributeKind::ss:
      for (auto& v : node->ss(name))
        attr->add_strings(v);
      break;
    case AttributeKind::t: {
      auto t = attr->mutable_t();
      if (use_external_data_format && !t->has_name()) {
        t->set_name(
            createAttributeTensorName(node_proto, t, name, num_external_data_));
      }
      EncodeTensor(
          t, node->t(name), {}, use_external_data_format, onnx_file_path);
    } break;
```
- **EN**: This chunk continues `AddAttribute` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Switch-based dispatch selects specialized behavior for each opcode, node kind, or schema case. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `AddAttribute`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 基于 switch 的分发为不同 opcode、节点类型或 schema 分支选择专门行为。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1112-1147
```cpp
    case AttributeKind::ts:
      for (auto& v : node->ts(name)) {
        auto t = attr->add_tensors();
        if (use_external_data_format && !t->has_name()) {
          t->set_name(createAttributeTensorName(
              node_proto, t, name, num_external_data_));
        }
        EncodeTensor(t, v, {}, use_external_data_format, onnx_file_path);
      }
      break;
    case AttributeKind::ty: {
      attr->set_type(onnx::AttributeProto_AttributeType_TYPE_PROTO);
      auto tp = attr->mutable_tp();
      const TypePtr& node_type = node->ty(name);
      EncodeTypeProto(
          tp, node_type, node_proto->op_type() + "_" + name.toDisplayString());
    } break;
    case AttributeKind::tys: {
      attr->set_type(onnx::AttributeProto_AttributeType_TYPE_PROTOS);
      size_t index = 0;
      for (auto& v : node->tys(name)) {
        auto tp = attr->add_type_protos();
        EncodeTypeProto(
            tp,
            v,
            node_proto->op_type() + "_" + name.toDisplayString() + "_" +
                std::to_string(index));
        index++;
      }
    } break;
    case AttributeKind::g: {
      auto g = attr->mutable_g();
      EncodeGraph(
          g,
          node->g(name),
          {},
```
- **EN**: This chunk continues `AddAttribute` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `AddAttribute`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1148-1174
```cpp
          {},
          true,
          true,
          use_external_data_format,
          onnx_file_path);
    } break;
    case AttributeKind::gs:
      for (auto& v : node->gs(name)) {
        auto g = attr->add_graphs();
        EncodeGraph(
            g, v, {}, {}, true, true, use_external_data_format, onnx_file_path);
      }
      break;
    default:
      std::ostringstream err_msg;
      err_msg << "attribute \"" << name.toDisplayString()
              << "\" has unexpected kind: " << toString(node->kindOf(name));
      throw std::runtime_error(err_msg.str());
  }
}

void GraphEncoder::AddAttribute(
    onnx::FunctionProto* func_proto,
    const std::string& name) {
  TORCH_INTERNAL_ASSERT(nullptr != func_proto);
  func_proto->add_attribute(name);
}
```
- **EN**: This chunk defines `AddAttribute`, which implements a focused step in loading or storing scripted programs. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `AddAttribute`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1176-1209
```cpp
void GraphEncoder::EncodeLocalFunctionOpsetImport(
    onnx::FunctionProto* func_proto,
    const Node* n,
    std::unordered_set<std::string>& custom_domains) {
  if (!n->kind().is_onnx()) {
    std::string domain;
    if (n->kind().is_aten() || n->kind().is_caffe2()) {
      domain = n->kind().domainString();
    } else { //  Custom namespace and domain
      domain = n->kind().ns().toUnqualString();
    }
    domains_.insert(domain);

    if (custom_domains.find(domain) == custom_domains.end()) {
      custom_domains.insert(domain);

      auto* custom_imp = func_proto->add_opset_import();
      custom_imp->set_domain(domain);
      //  Check if domain version is registered. If not, set to version 1
      auto it = custom_opsets_.find(domain);
      if (it == custom_opsets_.end())
        custom_imp->set_version(1);
      else {
        custom_imp->set_version(it->second);
      }
    }
  }

  for (auto* b : n->blocks()) {
    for (auto* sub_n : b->nodes()) {
      EncodeLocalFunctionOpsetImport(func_proto, sub_n, custom_domains);
    }
  }
}
```
- **EN**: This chunk defines `EncodeLocalFunctionOpsetImport`, which imports stored definitions into live runtime or compilation-unit objects. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `EncodeLocalFunctionOpsetImport`，其作用是把已存储定义导入为可用的运行时或编译单元对象。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1211-1243
```cpp
void GraphEncoder::EncodeLocalFunction(
    onnx::GraphProto* graph_proto,
    onnx::FunctionProto* func_proto,
    const Node* n,
    bool add_node_names,
    bool use_external_data_format,
    const std::string& onnx_file_path) {
  const auto fsub_g = n->g(Symbol::attr("graph"));
  func_proto->set_name(n->s(::c10::attr::name));

  for (auto input : fsub_g->inputs()) {
    func_proto->add_input(input->debugName());
  }
  for (auto output : fsub_g->outputs()) {
    func_proto->add_output(output->debugName());
  }

  // encode attributes names
  if (n->hasAttribute(Symbol::attr("attributes"))) {
    for (const auto& attr_name : n->ss(Symbol::attr("attributes"))) {
      AddAttribute(func_proto, attr_name);
    }
  }

  auto* imp = func_proto->add_opset_import();
  // This is the version of ONNX operator set we are targeting
  imp->set_version(onnx_opset_version_);

  // add for custom domain as well.
  const auto& domain = n->s(Symbol::attr("domain"));
  func_proto->set_domain(domain);
  domains_.insert(domain);
  std::unordered_set<std::string> custom_domains;
```
- **EN**: This chunk continues `EncodeLocalFunctionOpsetImport` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段延续了 `EncodeLocalFunctionOpsetImport`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1245-1277
```cpp
  for (auto* fsub_n : fsub_g->nodes()) {
    if (fsub_n->mustBeNone()) {
      // None nodes are used to implement optional inputs. One
      // way to "not provide" an optional input is to create an
      // Undefined node, and pass its output as that input.
      continue;
    }
    auto* n_proto = func_proto->add_node();
    EncodeNode(
        graph_proto,
        n_proto,
        fsub_n,
        add_node_names,
        use_external_data_format,
        onnx_file_path);
    EncodeLocalFunctionOpsetImport(func_proto, fsub_n, custom_domains);
  }
}

void GraphEncoder::EncodeTypeProto(
    onnx::TypeProto* type_proto,
    const TypePtr& node_type,
    const std::string& name) {
  if (TensorTypePtr tensor_type = node_type->cast<TensorType>()) {
    onnx::TypeProto_Tensor* onnx_tensor_type =
        type_proto->mutable_tensor_type();
    TensorTypeToONNXType(tensor_type, "", name, {}, onnx_tensor_type);
  } else if (ListTypePtr list_type = node_type->cast<ListType>()) {
    onnx::TypeProto_Sequence* seq_type = type_proto->mutable_sequence_type();
    auto elem_type = list_type->getElementType();
    EncodeTypeProto(seq_type->mutable_elem_type(), elem_type, name);
  }
}
```
- **EN**: This chunk defines `EncodeTypeProto`, which implements a focused step in loading or storing scripted programs. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 这一段定义了 `EncodeTypeProto`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1279-1314
```cpp
void GraphEncoder::EncodeTensor(
    onnx::TensorProto* tensor_proto,
    const at::Tensor& tensor,
    const std::optional<std::string>& external_ref,
    const bool use_external_data_format,
    const std::string& onnx_file_path) {
  for (auto d : tensor.sizes()) {
    tensor_proto->add_dims(d);
  }
  tensor_proto->set_data_type(ATenTypeToOnnxType(tensor.scalar_type()));
  at::Tensor t;
  // CPU's HalfTensor doesn't have contiguous(), so first calling contiguous()
  // TODO We don't call .cpu() on quantized tensors as it fails when calling
  // aten::empty() on quantized tensors beyond certain size. Issue #29435.
  if (tensor.is_quantized()) {
    t = tensor.contiguous();
  } else {
    t = tensor.contiguous().cpu();
  }

  // Either defer_weight_export should be true and external_ref must be present,
  // or use_external_data_format should be true, not both at the same time. They
  // can both be false at the same time (for ONNX export for regular model
  // size).
  TORCH_INTERNAL_ASSERT(
      !((defer_weight_export_ && external_ref) && use_external_data_format));
  // Add a buffer to the raw_data_export_map for the caller to dump into an
  // external data store. If external_ref is not specified, we instead dump
  // the contiguous data into the protobuf itself
  if (defer_weight_export_ && external_ref) {
    // For now, we use the name of the tensor as the external lookup name to
    // avoid ONNX protobuf changes.
    TORCH_INTERNAL_ASSERT(external_ref.value() == tensor_proto->name());
    TORCH_INTERNAL_ASSERT(
        raw_data_export_map_.count(external_ref.value()) == 0);
    raw_data_export_map_[external_ref.value()] = t;
```
- **EN**: This chunk defines `EncodeTensor`, which implements a focused step in loading or storing scripted programs. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段定义了 `EncodeTensor`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1315-1337
```cpp
    tensor_proto->set_raw_data("__EXTERNAL");
  } else {
    TORCH_INTERNAL_ASSERT(t.is_contiguous());
    size_t tensorSize = static_cast<size_t>(c10::multiply_integers(
        std::begin(tensor.sizes()), std::end(tensor.sizes())));
    if (use_external_data_format &&
        tensorSize > ParamSizeThresholdForExternalStorage) {
      TORCH_INTERNAL_ASSERT(!onnx_file_path.empty());
      TORCH_INTERNAL_ASSERT(tensor_proto->has_name());
      auto tensorName = GetExternalFileName(tensor_proto->name());
      CreateExternalFile(t, tensorName, onnx_file_path);
      onnx::StringStringEntryProto* location =
          tensor_proto->mutable_external_data()->Add();
      location->set_key("location");
      location->set_value(tensorName);
      tensor_proto->set_data_location(onnx::TensorProto_DataLocation_EXTERNAL);
    } else {
      // According to ParseData function's comments in onnx, tensor data is
      // always little endian.
      tensor_proto->set_raw_data(get_little_endian_data(t));
    }
  }
}
```
- **EN**: This chunk continues `EncodeTensor` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages.
- **CN**: 这一段延续了 `EncodeTensor`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。

### Lines 1339-1362
```cpp
void GraphEncoder::EncodeIntermediateValueInfo(
    onnx::GraphProto* graph_proto,
    const Value* v) {
  // Motivation is to encode ValueInfo for onnx local function nodes.
  auto n = v->node();
  if (n->kind().is_onnx() || n->kind().is_aten()) {
    // Encode value info only for non-onnx or non-ATen nodes.
    return;
  }
  if (n->owningGraph() != graph_.get()) {
    // Encode value info only for node in main graph.
    return;
  }
  for (const auto* o : graph_->outputs()) {
    // Do not encode value info for graph outputs.
    if (o == v) {
      return;
    }
  }
  auto v_info_p = graph_proto->add_value_info();
  EncodeValueInfo(graph_proto, v_info_p, v);
}

} // namespace
```
- **EN**: This chunk defines `EncodeIntermediateValueInfo`, which executes prepared logic against runtime values or IR state. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `EncodeIntermediateValueInfo`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1364-1393
```cpp
std::string pretty_print_onnx(
    const std::shared_ptr<Graph>& graph,
    const std::map<std::string, at::Tensor>& initializers,
    int64_t onnx_opset_version,
    bool defer_weight_export,
    ::torch::onnx::OperatorExportTypes operator_export_type,
    bool google_printer,
    bool keep_initializers_as_inputs,
    const std::map<std::string, int>& custom_opsets,
    bool add_node_names) {
  auto graph_encoder = GraphEncoder(
      graph,
      onnx_opset_version,
      operator_export_type,
      initializers,
      std::unordered_map<
          std::string,
          std::unordered_map<int64_t, std::string>>{},
      defer_weight_export,
      true,
      keep_initializers_as_inputs,
      custom_opsets,
      add_node_names,
      false,
      std::string());
  if (google_printer) {
    return graph_encoder.get_model_proto()->DebugString();
  }
  return prettyPrint(*graph_encoder.get_model_proto());
}
```
- **EN**: This chunk continues `EncodeIntermediateValueInfo` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `EncodeIntermediateValueInfo`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1395-1430
```cpp
std::tuple<
    std::shared_ptr<::ONNX_NAMESPACE::ModelProto>,
    RawDataExportMap,
    SymbolDimMap,
    bool,
    NodeNameMap>
export_onnx(
    const std::shared_ptr<Graph>& graph,
    const std::map<std::string, at::Tensor>& initializers,
    int64_t onnx_opset_version,
    const std::unordered_map<
        std::string,
        std::unordered_map<std::int64_t, std::string>>& dynamic_axes,
    bool defer_weight_export,
    ::torch::onnx::OperatorExportTypes operator_export_type,
    bool strip_doc_string,
    bool keep_initializers_as_inputs,
    const std::map<std::string, int>& custom_opsets,
    bool add_node_names,
    bool use_external_data_format,
    const std::string& onnx_file_path,
    const NodeAttrNameMap& node_attr_to_name) {
  auto graph_encoder = GraphEncoder(
      graph,
      onnx_opset_version,
      operator_export_type,
      initializers,
      dynamic_axes,
      defer_weight_export,
      strip_doc_string,
      keep_initializers_as_inputs,
      custom_opsets,
      add_node_names,
      use_external_data_format,
      onnx_file_path,
      node_attr_to_name);
```
- **EN**: This chunk continues `EncodeIntermediateValueInfo` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `EncodeIntermediateValueInfo`，进一步展开其内部控制流或数据流转。

### Lines 1431-1466
```cpp
  GRAPH_DEBUG("onnx proto:", prettyPrint(*graph_encoder.get_model_proto()));
  return std::make_tuple(
      graph_encoder.get_model_proto(),
      graph_encoder.get_raw_data_export_map(),
      graph_encoder.get_symbol_dim_param_map(),
      graph_encoder.get_use_external_data_format(),
      graph_encoder.get_onnx_node_names());
}

std::string serialize_model_proto_to_string(
    const std::shared_ptr<::ONNX_NAMESPACE::ModelProto>& model_proto) {
  return model_proto->SerializeAsString();
}

void check_onnx_proto(const std::string& proto_string) {
  onnx::ModelProto model;
  if (!ParseProtoFromBytes(&model, proto_string.c_str(), proto_string.size())) {
    throw std::runtime_error("Invalid ONNX proto string.");
  }
  // 1. baseline check
  // These two checks prevent broken graph being generated
  // And errors out exporting if that happens.
  onnx::checker::check_model(model);
  onnx::shape_inference::InferShapes(model);
  // 2. full check
  // apply strict mode shape type inference check which examines
  // whether it's a valid ONNX graph or not. As for some users, they
  // don't need a fully valid ONNX graph to run their model, we simply
  // add this information as warning message if it fails.
  try {
    auto* schema_registry = onnx::OpSchemaRegistry::Instance();
    onnx::ShapeInferenceOptions options{
        /*check_type_val=*/true,
        /*strict_mode_val=*/true};
    onnx::shape_inference::InferShapes(model, schema_registry, options);
  } catch (const onnx::InferenceError& ex) {
```
- **EN**: This chunk defines `runtime_error`, which executes prepared logic against runtime values or IR state. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `runtime_error`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1467-1477
```cpp
    TORCH_WARN(
        "The exported ONNX model failed ONNX shape inference. "
        "The model will not be executable by the ONNX Runtime. "
        "If this is unintended and you believe there is a bug, "
        "please report an issue at https://github.com/pytorch/pytorch/issues. "
        "Error reported by strict ONNX shape inference: ",
        ex.what());
  }
}

} // namespace torch::jit
```
- **EN**: This chunk continues `runtime_error` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `runtime_error`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **Serialization pipeline**
  - EN: Implements TorchScript import/export, bytecode, source loading, and serialized model interchange.
  - CN: 实现 TorchScript 的导入/导出、字节码、源码加载以及序列化模型交换。
- **GraphEncoder**
  - EN: `GraphEncoder` is a central symbol declared or implemented in this file.
  - CN: `GraphEncoder` 是本文件声明或实现的核心符号。
- **get_little_endian_data**
  - EN: `get_little_endian_data` is a central symbol declared or implemented in this file.
  - CN: `get_little_endian_data` 是本文件声明或实现的核心符号。
- **TorchScript runtime**
  - EN: Coordinates execution plans, stacks, and operator dispatch for scripted programs.
  - CN: 协调脚本程序的执行计划、栈以及算子分发。
- **Graph IR**
  - EN: Works on graph, node, and block structures that represent TorchScript programs.
  - CN: 围绕表示 TorchScript 程序的 graph、node 与 block 结构工作。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/serialization/export.h`, `torch/csrc/autograd/symbolic.h`, `torch/csrc/jit/jit_log.h`, `torch/csrc/jit/passes/dead_code_elimination.h`, `torch/csrc/jit/passes/inliner.h`, `torch/csrc/jit/runtime/instruction.h`, `torch/csrc/jit/serialization/import_export_constants.h`, `torch/csrc/jit/serialization/import_export_functions.h`, `torch/csrc/jit/serialization/import_export_helpers.h`, `torch/csrc/jit/serialization/onnx.h`, `torch/csrc/jit/serialization/pickler.h`, `torch/csrc/onnx/back_compat.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/ATen.h`, `ATen/Utils.h`, `ATen/core/functional.h`, `c10/macros/Macros.h`, `c10/util/Exception.h`, `c10/util/accumulate.h`, `c10/util/irange.h`
- **Third-party libraries / 第三方库**: `onnx/checker.h`, `onnx/onnx_pb.h`, `onnx/proto_utils.h`, `onnx/shape_inference/implementation.h`
- **Standard library / 标准库**: `memory`, `optional`, `regex`, `set`, `sstream`, `string`, `utility`, `vector`
- **Primary symbols in this file / 本文件核心符号**: `GraphEncoder`, `get_little_endian_data`, `writeArchiveAndTensors`, `getNodeStackTraceString`, `validateBlock`, `runtime_error`, `validateGraph`, `GetFileRootPath`
- **Note / 说明**: 33 direct includes were detected; only the first few are listed above for readability. / 检测到 33 个直接包含，为便于阅读这里只列出前若干项。

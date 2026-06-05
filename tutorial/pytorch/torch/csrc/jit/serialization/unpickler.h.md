# unpickler.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/serialization/unpickler.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Serializes IValues, module state, or bytecode into portable binary/container representations.
- **Purpose (CN)**: 把 IValue、模块状态或字节码序列化为可移植的二进制/容器表示。
## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
#pragma once

#include <ATen/core/ivalue.h>
#include <c10/util/ArrayRef.h>
#include <caffe2/serialize/inline_container.h>

#include <torch/csrc/Export.h>
#include <torch/csrc/jit/frontend/script_type_parser.h>
#include <torch/csrc/jit/serialization/pickler_helper.h>

namespace torch::jit {

using TypeResolver =
    std::function<c10::StrongTypePtr(const c10::QualifiedName&)>;
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/Export.h, torch/csrc/jit/frontend/script_type_parser.h, torch/csrc/jit/serialization/pickler_helper.h; ATen/c10 facilities such as ATen/core/ivalue.h, c10/util/ArrayRef.h; standard-library headers such as caffe2/serialize/inline_container.h. The preprocessor guard keeps declarations single-instanced when the header is included transitively. The namespace declarations place the code inside torch::jit, matching the surrounding JIT subsystem.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/Export.h、torch/csrc/jit/frontend/script_type_parser.h、torch/csrc/jit/serialization/pickler_helper.h；ATen/c10 基础设施，如 ATen/core/ivalue.h、c10/util/ArrayRef.h；标准库头文件，如 caffe2/serialize/inline_container.h。 预处理器保护用于避免头文件在传递包含时被重复展开。 命名空间声明把代码放入 torch::jit 中，与周边 JIT 子系统保持一致。

### Lines 16-28
```cpp
using ObjLoader = std::function<
    c10::intrusive_ptr<c10::ivalue::Object>(const at::StrongTypePtr&, IValue)>;

class DeserializationStorageContext;

// [unpickler refactor] there is some cruft around PickleOpCode::BUILD,
// PickleOpCode::NEWOBJ, and the last_opcode_ member below that should be
// deleted at some point, the Pickler doesn't produce it and it's only around to
// support models saved before 1.1
class TORCH_API Unpickler {
  AT_DISALLOW_COPY_AND_ASSIGN(Unpickler);

  using TypeParserT = c10::TypePtr (*)(const std::string&);
```
- **EN**: It introduces or extends DeserializationStorageContext, TORCH_API, which define the primary data structures or interfaces for this portion of the file.
- **CN**: 它引入或扩展了 DeserializationStorageContext、TORCH_API，这些类型定义了本段涉及的主要数据结构或接口。

### Lines 30-45
```cpp
 public:
  // tensors inside the pickle are references to the tensor_table.
  // class_resolver is to resolve strong class type, type_resolver_ is
  // to resolve any JIT type. class_resolver and type_resolver are not merged
  // here because some use cases need to get strong class type that
  // type_resolver_ can not return.
  // NOLINTNEXTLINE(cppcoreguidelines-pro-type-member-init)
  Unpickler(
      std::function<size_t(char*, size_t)> reader,
      TypeResolver type_resolver,
      c10::ArrayRef<at::Tensor> tensor_table,
      TypeParserT type_parser = defaultTypeParser)
      : reader_(std::move(reader)),
        tensor_table_(tensor_table),
        type_resolver_(std::move(type_resolver)),
        use_storage_device_(false),
```
- **EN**: This chunk continues `TORCH_API` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `TORCH_API`，进一步展开其内部控制流或数据流转。

### Lines 46-61
```cpp
        type_parser_(type_parser),
        version_(caffe2::serialize::kProducedFileFormatVersion) {}

  // NOLINTNEXTLINE(cppcoreguidelines-pro-type-member-init)
  Unpickler(
      std::function<size_t(char*, size_t)> reader,
      TypeResolver type_resolver,
      c10::ArrayRef<at::Tensor> tensor_table,
      ObjLoader obj_loader,
      TypeParserT type_parser = defaultTypeParser)
      : reader_(std::move(reader)),
        tensor_table_(tensor_table),
        type_resolver_(std::move(type_resolver)),
        obj_loader_(std::move(obj_loader)),
        use_storage_device_(false),
        type_parser_(type_parser),
```
- **EN**: This chunk continues `TORCH_API` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `TORCH_API`，进一步展开其内部控制流或数据流转。

### Lines 62-77
```cpp
        version_(caffe2::serialize::kProducedFileFormatVersion) {}

  // tensors inside the pickle contain meta-data, the raw tensor
  // dead is retrieved by calling `read_record`.
  // NOLINTNEXTLINE(cppcoreguidelines-pro-type-member-init)
  Unpickler(
      std::function<size_t(char*, size_t)> reader,
      TypeResolver type_resolver,
      ObjLoader obj_loader,
      std::function<at::DataPtr(const std::string&)> read_record,
      std::optional<at::Device> device,
      bool use_storage_device = false,
      TypeParserT type_parser = defaultTypeParser,
      std::shared_ptr<DeserializationStorageContext> storage_context = nullptr)
      : reader_(std::move(reader)),
        type_resolver_(std::move(type_resolver)),
```
- **EN**: This chunk continues `TORCH_API` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `TORCH_API`，进一步展开其内部控制流或数据流转。

### Lines 78-88
```cpp
        obj_loader_(std::move(obj_loader)),
        read_record_(std::move(read_record)),
        device_(device),
        use_storage_device_(use_storage_device),
        type_parser_(type_parser),
        storage_context_(std::move(storage_context)),
        version_(caffe2::serialize::kProducedFileFormatVersion) {}

  Unpickler(Unpickler&&) = delete;
  Unpickler& operator=(Unpickler&&) = delete;
  ~Unpickler() = default;
```
- **EN**: This chunk continues `TORCH_API` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `TORCH_API`，进一步展开其内部控制流或数据流转。

### Lines 90-105
```cpp
  // consume the pickle stream, producing an IValue from the contents.
  // Type Tags: the pickler will restore the type tags on
  // List and Dict objects when possible IValue is an Object.
  // Otherwise, Dict and List objects will end up with Any as their tag.
  // If you know the type of the ivalue, tags can be restored with
  // restoreAccurateTypeTags
  IValue parse_ivalue();

  // [type tag serialization]
  // This is used to determine whether to restore type tags be recursively
  // descending into the returned stack object (if version_number <= 2), or
  // if version_number >= 3, to use the type strings included in the pickle
  // archive for container types. By default this is set to
  // `kProducedFileFormatVersion` so unless you're loading a pickle file
  // from alongside a corresponding `version` file, you don't need to set
  // the version manually.
```
- **EN**: This chunk declares `parse_ivalue`, which parses source text or schema-like input into internal data structures.
- **CN**: 这一段声明了 `parse_ivalue`，其作用是把源码文本或类 schema 输入解析为内部数据结构。

### Lines 106-121
```cpp
  void set_version(uint64_t version_number) {
    version_ = version_number;
  }

  static c10::TypePtr defaultTypeParser(const std::string& str) {
    ScriptTypeParser parser;
    return parser.parseType(str);
  }

 private:
  // No arguments ensures that a template argument must be specified
  // so that the number of bytes read / type read is explicit
  template <typename T>
  T read() {
    T item;
    if (sizeof(T) <= buffer_remaining_) {
```
- **EN**: This chunk defines `read`, which implements a focused step in loading or storing scripted programs. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `read`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 122-133
```cpp
      // Fast path: entirely from buffer.
      memcpy(&item, buffer_.data() + buffer_pos_, sizeof(T));
      buffer_remaining_ -= sizeof(T);
      buffer_pos_ += sizeof(T);
    } else {
      // Don't over-template the slow path, to avoid code size bloat.
      readSlowWithBuffer(reinterpret_cast<char*>(&item), sizeof(T));
    }
    return item;
  }
  void readSlowWithBuffer(char* dest, size_t sz);
  std::string readBytes(size_t num_bytes);
```
- **EN**: This chunk defines `readBytes`, which implements a focused step in loading or storing scripted programs. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `readBytes`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 135-150
```cpp
  double readFloat();
  void readGlobal(
      const std::string& module_name,
      const std::string& class_name);
  void rebuildTensor(bool quantized);
  void rebuildParameter();
  void rebuildTensorFromTypeV2();
  void rebuildSparseTensor();
#ifdef USE_DISTRIBUTED
  void rebuildRRef();
#endif
  PickleOpCode readInstruction();
  PickleOpCode readOpCode() {
    return static_cast<PickleOpCode>(read<uint8_t>());
  }
  std::string readString();
```
- **EN**: This chunk defines `readString`, which implements a focused step in loading or storing scripted programs. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `readString`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 151-164
```cpp
  void readList(IValue list_ivalue);
  void readListElements(IValue list_ivalue, size_t start);
  void setInput(size_t memo_id);
  void run();

  // Returns the number of bytes read. This should statefully
  // remember the position. Don't call reader_ directly.
  std::function<size_t(char*, size_t)> reader_;
  // Small buffer to avoid calling reader_ on a per-byte basis.
  std::array<char, 256> buffer_;
  size_t buffer_pos_{0};
  size_t buffer_remaining_{0};

  std::vector<IValue> stack_;
```
- **EN**: This chunk defines `run`, which executes prepared logic against runtime values or IR state.
- **CN**: 这一段定义了 `run`，其作用是针对运行时数值或 IR 状态执行准备好的逻辑。

### Lines 166-177
```cpp
  // globals are represented on the stack as IValue integer indices
  // into this list
  std::vector<std::function<void(void)>> globals_;
  std::vector<IValue> memo_table_;
  std::vector<size_t> marks_;
  c10::ArrayRef<at::Tensor> tensor_table_;

  // When deserializing types on lists and dicts, cache the type here
  // so we don't have to parse the same type multiple times. Strings
  // are already de-duplicated and replaced with BINGETs in the
  // pickler, so we can just use the actual data pointer of each string.
  std::unordered_map<std::string, c10::TypePtr> type_cache_;
```
- **EN**: This chunk continues `run` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `run`，进一步展开其内部控制流或数据流转。

### Lines 179-191
```cpp
  // optionally nullptr, needs to be present for creating classes
  TypeResolver type_resolver_;
  ObjLoader obj_loader_;
  IValue empty_tuple_;

  std::function<at::DataPtr(const std::string&)> read_record_;
  std::optional<at::Device> device_;
  // When set to true, Unpickler will ignore the pickled device and use the
  // device of the DataPtr returned by the read_record_ function. The default
  // value of this flag is false.
  const bool use_storage_device_;

  TypeParserT type_parser_{defaultTypeParser};
```
- **EN**: This chunk continues `run` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `run`，进一步展开其内部控制流或数据流转。

### Lines 193-206
```cpp
  // Used for torch.package to enable sharing of storages across
  // ScriptModules and eager modules
  std::shared_ptr<DeserializationStorageContext> storage_context_;

  // See [type tag serialization]
  uint64_t version_;

  // See [NOTE] skip_next_read_global
  uint8_t skip_next_read_global = 0;
};

void restoreAccurateTypeTags(const IValue& root, const c10::TypePtr& type_tag);

} // namespace torch::jit
```
- **EN**: This chunk declares `restoreAccurateTypeTags`, which implements a focused step in loading or storing scripted programs.
- **CN**: 这一段声明了 `restoreAccurateTypeTags`，其作用是实现脚本程序加载或存储过程中的一个关键步骤。

## Key Concepts / 关键概念

- **Serialization pipeline**
  - EN: Implements TorchScript import/export, bytecode, source loading, and serialized model interchange.
  - CN: 实现 TorchScript 的导入/导出、字节码、源码加载以及序列化模型交换。
- **DeserializationStorageContext**
  - EN: `DeserializationStorageContext` is a central symbol declared or implemented in this file.
  - CN: `DeserializationStorageContext` 是本文件声明或实现的核心符号。
- **TORCH_API**
  - EN: `TORCH_API` is a central symbol declared or implemented in this file.
  - CN: `TORCH_API` 是本文件声明或实现的核心符号。
- **IValue data model**
  - EN: Moves values through generic boxed runtime containers used by TorchScript.
  - CN: 通过 TorchScript 使用的通用装箱运行时容器传递数值。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **Pickle-style encoding**
  - EN: Encodes values and object graphs into compact binary representations.
  - CN: 把值和对象图编码为紧凑的二进制表示。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/Export.h`, `torch/csrc/jit/frontend/script_type_parser.h`, `torch/csrc/jit/serialization/pickler_helper.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/core/ivalue.h`, `c10/util/ArrayRef.h`
- **Standard library / 标准库**: `caffe2/serialize/inline_container.h`
- **Primary symbols in this file / 本文件核心符号**: `DeserializationStorageContext`, `TORCH_API`, `parse_ivalue`, `set_version`, `defaultTypeParser`, `read`, `readSlowWithBuffer`, `readBytes`

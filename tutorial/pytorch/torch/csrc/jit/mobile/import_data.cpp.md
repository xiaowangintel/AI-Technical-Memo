# import_data.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/mobile/import_data.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements loading, execution, optimization, or serialization support for the mobile / lite interpreter runtime. This specific file centers on `import_data.cpp`. It is tailored for lite interpreter or mobile deployment constraints.
- **Purpose (CN)**: 实现移动端 / 轻量解释器运行时的加载、执行、优化或序列化支持。 该文件具体围绕 `import_data.cpp` 展开。 其设计面向轻量解释器或移动端部署约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#include <torch/csrc/jit/mobile/import_data.h>

#include <ATen/Functions.h>
#include <ATen/core/ivalue.h>
#include <c10/util/irange.h>

#include <torch/csrc/jit/api/compilation_unit.h>
#include <torch/csrc/jit/mobile/file_format.h>
#include <torch/csrc/jit/mobile/flatbuffer_loader.h>
#include <torch/csrc/jit/mobile/import.h>
#include <torch/csrc/jit/mobile/import_export_common.h>
#include <torch/csrc/jit/mobile/module.h>
#include <torch/csrc/jit/mobile/observer.h>
#include <torch/csrc/jit/mobile/type_parser.h>
#include <torch/csrc/jit/runtime/instruction.h>
#include <torch/csrc/jit/serialization/unpickler.h>
#include <torch/custom_class.h>

#include <caffe2/serialize/in_memory_adapter.h>
#include <string>
#include <vector>

namespace torch::jit {
using caffe2::serialize::PyTorchStreamReader;

namespace {

/**
```

- **EN:** This range collects the headers or imports needed by the following declarations and implementation blocks.
- **CN:** 这一段集中引入后续声明与实现所需的头文件或模块。
- **EN:** It enters or references namespace scopes such as torch::jit, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。

### Lines 29-56 / 第 29-56 行

```cpp
 * Given a ZIP file containing a file named "data.pkl", uses Pickle to
 * deserialize the file and returns the IValue inside it.
 */
class IValueUnpickler final {
 public:
  explicit IValueUnpickler(std::unique_ptr<PyTorchStreamReader> reader);
  c10::IValue deserialize(std::optional<at::Device> device);

 private:
  c10::IValue readArchive(
      const std::string& archive_name,
      std::shared_ptr<mobile::CompilationUnit> mcu,
      std::optional<at::Device> device);

  std::shared_ptr<CompilationUnit> compilation_unit_;
  std::unique_ptr<PyTorchStreamReader> reader_;
};

IValueUnpickler::IValueUnpickler(std::unique_ptr<PyTorchStreamReader> reader)
    : compilation_unit_(std::make_shared<CompilationUnit>()),
      reader_(std::move(reader)) {}

c10::IValue IValueUnpickler::deserialize(std::optional<at::Device> device) {
  auto mcu = std::make_shared<mobile::CompilationUnit>();

  return readArchive("data", mcu, device);
}

```

- **EN:** The block declares or refines core types including IValueUnpickler.
- **CN:** 该代码块声明或细化了 IValueUnpickler 等核心类型。
- **EN:** Important callable entry points in this range include IValueUnpickler, deserialize, readArchive.
- **CN:** 这一段的重要可调用入口包括 IValueUnpickler, deserialize, readArchive。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。

### Lines 57-84 / 第 57-84 行

```cpp
c10::IValue IValueUnpickler::readArchive(
    const std::string& archive_name,
    std::shared_ptr<mobile::CompilationUnit> mcu,
    std::optional<at::Device> device) {
  std::stringstream picklename;
  picklename << archive_name << ".pkl";
  at::DataPtr pickle_ptr;
  size_t pickle_size = 0;
  std::tie(pickle_ptr, pickle_size) = reader_->getRecord(picklename.str());

  size_t bytes_read = 0;
  auto data = reinterpret_cast<const char*>(pickle_ptr.get());
  auto reader = [&](char* buffer, size_t len) -> size_t {
    if (bytes_read >= pickle_size) {
      return 0;
    }
    len = std::min(pickle_size - bytes_read, len);
    // Copy len bytes into buffer
    const char* start = data + bytes_read;
    std::memcpy(buffer, start, len);
    bytes_read += len;
    return len;
  };

  static const c10::QualifiedName torchPrefix = "__torch__";
  auto type_resolver = [&](const c10::QualifiedName& qn) {
    TypePtr type;
    // HACK: first we check whether the name starts with `__torch__` to tell if
```

- **EN:** Important callable entry points in this range include readArchive, tie, memcpy.
- **CN:** 这一段的重要可调用入口包括 readArchive, tie, memcpy。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 85-112 / 第 85-112 行

```cpp
    // it's "supposed" to be a class type. This is a reliable check today, but
    // there is no guarantee that this is the case. The real solution is to
    // merge type parsers so we can share class resolution logic.
    if (torchPrefix.isPrefixOf(qn)) {
      if (compilation_unit_->get_class(qn) == nullptr) {
        auto typeptr = ClassType::create(qn, compilation_unit_, true);
        compilation_unit_->register_type(typeptr);
      }
      type = compilation_unit_->get_class(qn);
    } else {
      type = c10::parseType(qn.qualifiedName());
    }
    return c10::StrongTypePtr(compilation_unit_, type);
  };

  auto obj_loader = [&](const at::StrongTypePtr& type, IValue input) {
    auto cls = type.type_->expect<at::ClassType>();
    auto qn = cls->name();
    c10::QualifiedName method_name(qn.value(), "__setstate__");
    auto setstate = mcu->find_function(method_name);
    auto find_custom_class_with_setstate = [&qn]() -> c10::ClassTypePtr {
      auto custom_class_type = torch::jit::getCustomClass(qn->qualifiedName());
      if (custom_class_type && custom_class_type->findMethod("__setstate__")) {
        return custom_class_type;
      }
      return nullptr;
    };
    if (setstate) {
```

- **EN:** The block declares or refines core types including type, resolution.
- **CN:** 该代码块声明或细化了 type, resolution 等核心类型。
- **EN:** Important callable entry points in this range include StrongTypePtr, method_name.
- **CN:** 这一段的重要可调用入口包括 StrongTypePtr, method_name。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。

### Lines 113-140 / 第 113-140 行

```cpp
      auto obj = c10::ivalue::Object::create(type, 0);
      Stack stack({obj, input});
      setstate->run(stack);
      return obj;
    } else if (auto custom_class_type = find_custom_class_with_setstate()) {
      auto obj = c10::ivalue::Object::create(
          c10::StrongTypePtr(nullptr, custom_class_type), 1);
      Stack stack({obj, input});
      custom_class_type->getMethod("__setstate__").run(stack);
      return obj;
    } else {
      auto dict = std::move(input).toGenericDict();
      size_t ndict = dict.size();
      auto obj = c10::ivalue::Object::create(type, ndict);
      auto it = dict.begin();
      for (const auto i : c10::irange(ndict)) {
        std::stringstream name;
        name << it->key();
        cls->addOrCheckAttribute(name.str(), it->key().type());
        obj->setSlot(i, it->value());
        ++it;
      }
      return obj;
    }
  };

  auto read_record = [&](const std::string& name) {
    std::stringstream ss;
```

- **EN:** Important callable entry points in this range include StrongTypePtr.
- **CN:** 这一段的重要可调用入口包括 StrongTypePtr。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 141-168 / 第 141-168 行

```cpp
    ss << archive_name << '/' << name;
    return std::get<0>(reader_->getRecord(ss.str()));
  };

  Unpickler unpickler(
      reader,
      std::move(type_resolver),
      std::move(obj_loader),
      std::move(read_record),
      device,
      false,
      nullptr);
  return unpickler.parse_ivalue();
}

/**
 * Extracts and returns the parameter map serialized as ZIP + Pickle in @p rai.
 */
std::map<std::string, at::Tensor> load_parameters_from_zip(
    std::unique_ptr<ReadAdapterInterface> rai,
    std::optional<c10::Device> device) {
  auto reader = std::make_unique<PyTorchStreamReader>(std::move(rai));
  IValueUnpickler unpickler(std::move(reader));
  auto result = unpickler.deserialize(device).toGenericDict();
  std::map<std::string, at::Tensor> map;
  for (const auto& e : result) {
    auto key = e.key().toStringRef();
    auto value = e.value().toTensor().tensor_data();
```

- **EN:** Important callable entry points in this range include unpickler, load_parameters_from_zip.
- **CN:** 这一段的重要可调用入口包括 unpickler, load_parameters_from_zip。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 169-196 / 第 169-196 行

```cpp
    map[key] = value;
  }
  return map;
}

} // namespace

/**
 * Extracts the parameter map stored in @p module. Expects a layout
 * compatible with the one created by #_save_parameters().
 */
std::map<std::string, at::Tensor> mobile_module_to_parameter_map(
    const mobile::Module& module) {
  // Safely look for a slot with the expected name. Note that
  // c10::ivalue::Object::getAttr() is not safe if the attribute isn't present.
  auto obj = module._ivalue();
  const std::vector<IValue>& slots = obj->slots();
  for (const auto i : c10::irange(slots.size())) {
    if (obj->type()->getAttributeName(i) ==
        mobile::internal::kSavedParametersAttributeName) {
      // Found a slot with the right name; make sure it's a
      // Dict<string, Tensor>.
      c10::IValue data = slots[i];
      if (data.isGenericDict()) {
        auto data_dict = data.toGenericDict();

        // The key and value should be DynamicTypes that wrap String and Tensor.
        c10::DynamicType* keyType =
```

- **EN:** Important callable entry points in this range include mobile_module_to_parameter_map.
- **CN:** 这一段的重要可调用入口包括 mobile_module_to_parameter_map。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 197-224 / 第 197-224 行

```cpp
            data_dict.keyType()->castRaw<c10::DynamicType>();
        c10::DynamicType* valueType =
            data_dict.valueType()->castRaw<c10::DynamicType>();
        if (keyType != nullptr &&
            keyType->fallback()->kind() == TypeKind::StringType &&
            valueType != nullptr &&
            valueType->fallback()->kind() == TypeKind::TensorType) {
          // Name and type are good; copy the contents to the output map.
          std::map<std::string, at::Tensor> params;
          for (const auto& e : data_dict) {
            // The source Tensor points into the flatbuffer data associated with
            // the Module. But, this Tensor needs to outlive the Module, since
            // the caller of _load_parameters() won't have a pointer to the
            // Module. So, return a deep copy.
            const auto& source = e.value().toTensor();
            at::Tensor copy = at::empty_like(source); // Must be the same shape.
            copy.copy_(source);

            params[e.key().toStringRef()] = copy;
          }
          return params;
        }
      }
    }
  }

  TORCH_CHECK(
      false,
```

- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 225-252 / 第 225-252 行

```cpp
      "Could not find Dict<string, Tensor> named '",
      mobile::internal::kSavedParametersAttributeName,
      "' in deserialized mobile::Module");
}

static std::map<std::string, at::Tensor> _load_parameters_bytes(
    const std::shared_ptr<char>& data,
    size_t size,
    std::optional<at::Device> device) {
  TORCH_CHECK(size >= kFileFormatHeaderSize, "Unrecognized data format");
  FileFormat format = getFileFormat(data.get());
  // Call the appropriate parser.
  std::map<std::string, at::Tensor> map;
  switch (format) {
    case FileFormat::FlatbufferFileFormat: {
      auto m = parse_flatbuffer_no_object(data, size, device);
      map = mobile_module_to_parameter_map(m);
      break;
    }

    case FileFormat::ZipFileFormat: {
      auto rai = std::make_unique<caffe2::serialize::MemoryReadAdapter>(
          data.get(), size);
      map = load_parameters_from_zip(std::move(rai), device);
      break;
    }

    default:
```

- **EN:** Important callable entry points in this range include _load_parameters_bytes, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 _load_parameters_bytes, TORCH_CHECK。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Module API / 模块 API, Serialization / 序列化, Mobile runtime / 移动端运行时, Alias analysis / 别名分析, Declared symbols / 声明的符号, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Module API / 模块 API, Serialization / 序列化, Mobile runtime / 移动端运行时, Alias analysis / 别名分析, Declared symbols / 声明的符号, Branching logic / 分支逻辑。

### Lines 253-272 / 第 253-272 行

```cpp
      TORCH_CHECK(false, "Unrecognized data format");
  }
  return map;
}

std::map<std::string, at::Tensor> _load_parameters(
    std::istream& in,
    std::optional<at::Device> device) {
  auto [data, size] = get_stream_content(in);
  return _load_parameters_bytes(data, size, device);
}

std::map<std::string, at::Tensor> _load_parameters(
    const std::string& filename,
    std::optional<at::Device> device) {
  auto [data, size] = get_file_content(filename.c_str());
  return _load_parameters_bytes(data, size, device);
}

} // namespace torch::jit
```

- **EN:** Important callable entry points in this range include TORCH_CHECK, _load_parameters, _load_parameters_bytes.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK, _load_parameters, _load_parameters_bytes。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Operator schema / 算子模式, Declared symbols / 声明的符号, Namespace scoping / 命名空间作用域, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Operator schema / 算子模式, Declared symbols / 声明的符号, Namespace scoping / 命名空间作用域, Result propagation / 结果传递。

## Key Concepts / 关键概念

- **Lite interpreter runtime** — 轻量解释器运行时
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Serialization** — 序列化
- **Mobile runtime** — 移动端运行时
- **Python binding** — Python 绑定
- **Registration** — 注册机制
- **Alias analysis** — 别名分析

## Dependencies / 依赖关系

- `torch/csrc/jit/mobile/import_data.h`
- `ATen/Functions.h`
- `ATen/core/ivalue.h`
- `c10/util/irange.h`
- `torch/csrc/jit/api/compilation_unit.h`
- `torch/csrc/jit/mobile/file_format.h`
- `torch/csrc/jit/mobile/flatbuffer_loader.h`
- `torch/csrc/jit/mobile/import.h`
- `torch/csrc/jit/mobile/import_export_common.h`
- `torch/csrc/jit/mobile/module.h`
- `torch/csrc/jit/mobile/observer.h`
- `torch/csrc/jit/mobile/type_parser.h`
- `torch/csrc/jit/runtime/instruction.h`
- `torch/csrc/jit/serialization/unpickler.h`
- `torch/custom_class.h`

# AsmState.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/IR/AsmState.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Declares C++ interfaces and helper types for the MLIR AsmState component. The leading comments describe it as: This file defines various classes and utilites for interacting with the MLIR.
- **用途（CN）**: 声明 MLIR AsmState 组件相关的 C++ 接口与辅助类型。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
````cpp
//===- AsmState.h - Assembly State Utilities --------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines various classes and utilites for interacting with the MLIR
// assembly formats.
//
//===----------------------------------------------------------------------===//

#ifndef MLIR_IR_ASMSTATE_H_
#define MLIR_IR_ASMSTATE_H_

#include "mlir/Bytecode/BytecodeReaderConfig.h"
#include "mlir/IR/OperationSupport.h"
#include "mlir/Support/LLVM.h"
#include "llvm/ADT/MapVector.h"
#include "llvm/ADT/StringMap.h"

#include <memory>
#include <variant>
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 26-33
````cpp
namespace mlir {
class AsmResourcePrinter;
class AsmDialectResourceHandle;
class Operation;

namespace detail {
class AsmStateImpl;
} // namespace detail
````
- **EN**: This C++ declaration introduces `AsmResourcePrinter` and establishes part of the API surface for `AsmState`.
- **CN**: 该 C++ 声明引入了 `AsmResourcePrinter`，并构成 `AsmState` API 表面的一部分。

### Lines 35-80
````cpp
//===----------------------------------------------------------------------===//
// Resources
//===----------------------------------------------------------------------===//

/// The following classes enable support for parsing and printing resources
/// within MLIR assembly formats. Resources are a mechanism by which dialects,
/// and external clients, may attach additional information when parsing or
/// printing IR without that information being encoded in the IR itself.
/// Resources are not uniqued within the MLIR context, are not attached directly
/// to any operation, and are solely intended to live and be processed outside
/// of the immediate IR.
///
/// Resources are encoded using a key-value pair nested within dictionaries
/// anchored either on a dialect, or an externally registered entity.
/// Dictionaries anchored on dialects use the dialect namespace directly, and
/// dictionaries anchored on external entities use a provided unique identifier.
/// The resource key is an identifier used to disambiguate the data. The
/// resource value may be stored in various limited forms, but general encodings
/// use a string (human readable) or blob format (binary). Within the textual
/// format, an example may be of the form:
///
/// {-#
///   // The `dialect_resources` section within the file-level metadata
///   // dictionary is used to contain any dialect resource entries.
///   dialect_resources: {
///     // Here is a dictionary anchored on "foo_dialect", which is a dialect
///     // namespace.
///     foo_dialect: {
///       // `some_dialect_resource` is a key to be interpreted by the dialect,
///       // and used to initialize/configure/etc.
///       some_dialect_resource: "Some important resource value"
///     }
///   },
///   // The `external_resources` section within the file-level metadata
///   // dictionary is used to contain any non-dialect resource entries.
///   external_resources: {
///     // Here is a dictionary anchored on "mlir_reproducer", which is an
///     // external entity representing MLIR's crash reproducer functionality.
///     mlir_reproducer: {
///       // `pipeline` is an entry that holds a crash reproducer pipeline
///       // resource.
///       pipeline: "func.func(canonicalize,cse)"
///     }
///   }
/// #-}
///
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 81-95
````cpp
//===----------------------------------------------------------------------===//
// Resource Entry
//===----------------------------------------------------------------------===//

class HeapAsmResourceBlob;

/// This class represents a processed binary blob of data. A resource blob is
/// essentially a collection of data, potentially mutable, with an associated
/// deleter function (used if the data needs to be destroyed).
class AsmResourceBlob {
public:
  /// A deleter function that frees a blob given the data, allocation size, and
  /// allocation aligment.
  using DeleterFn =
      llvm::unique_function<void(void *data, size_t size, size_t align)>;
````
- **EN**: This C++ declaration introduces `HeapAsmResourceBlob` and establishes part of the API surface for `AsmState`. Representative entry points here include `void`.
- **CN**: 该 C++ 声明引入了 `HeapAsmResourceBlob`，并构成 `AsmState` API 表面的一部分。 这一段可见的代表性接口包括 `void`。

### Lines 98-135
````cpp
  //===--------------------------------------------------------------------===//
  // Construction
  //===--------------------------------------------------------------------===//

  AsmResourceBlob() = default;
  AsmResourceBlob(ArrayRef<char> data, size_t dataAlignment, DeleterFn deleter,
                  bool dataIsMutable)
      : data(data), dataAlignment(dataAlignment), deleter(std::move(deleter)),
        dataIsMutable(dataIsMutable) {}
  /// Utility constructor that initializes a blob with a non-char type T.
  template <typename T, typename DelT>
  AsmResourceBlob(ArrayRef<T> data, DelT &&deleteFn, bool dataIsMutable)
      : data((const char *)data.data(), data.size() * sizeof(T)),
        dataAlignment(alignof(T)),
        deleter([deleteFn = std::forward<DelT>(deleteFn)](
                    void *data, size_t size, size_t align) {
          return deleteFn((T *)data, size, align);
        }),
        dataIsMutable(dataIsMutable) {}
  AsmResourceBlob(AsmResourceBlob &&) = default;
  AsmResourceBlob &operator=(AsmResourceBlob &&rhs) {
    // Delete the current blob if necessary.
    if (deleter)
      deleter(const_cast<char *>(data.data()), data.size(), dataAlignment);

    // Take the data entries from rhs.
    data = rhs.data;
    dataAlignment = rhs.dataAlignment;
    deleter = std::move(rhs.deleter);
    dataIsMutable = rhs.dataIsMutable;
    return *this;
  }
  AsmResourceBlob(const AsmResourceBlob &) = delete;
  AsmResourceBlob &operator=(const AsmResourceBlob &) = delete;
  ~AsmResourceBlob() {
    if (deleter)
      deleter(const_cast<char *>(data.data()), data.size(), dataAlignment);
  }
````
- **EN**: This block groups callable interfaces such as `AsmResourceBlob`, `data`, `dataAlignment`, `deleter`, indicating how `AsmState` is queried or updated.
- **CN**: 该代码块聚合了 `AsmResourceBlob`, `data`, `dataAlignment`, `deleter` 等可调用接口，展示了如何查询或更新 `AsmState`。

### Lines 137-182
````cpp
  //===--------------------------------------------------------------------===//
  // Data Access
  //===--------------------------------------------------------------------===//

  /// Return the alignment of the underlying data.
  size_t getDataAlignment() const { return dataAlignment; }

  /// Return the raw underlying data of this blob.
  ArrayRef<char> getData() const { return data; }

  /// Return the underlying data as an array of the given type. This is an
  /// inherrently unsafe operation, and should only be used when the data is
  /// known to be of the correct type.
  template <typename T>
  ArrayRef<T> getDataAs() const {
    return llvm::ArrayRef<T>((const T *)data.data(), data.size() / sizeof(T));
  }

  /// Return a mutable reference to the raw underlying data of this blob.
  /// Asserts that the blob `isMutable`.
  MutableArrayRef<char> getMutableData() {
    assert(isMutable() &&
           "cannot access mutable reference to non-mutable data");
    return MutableArrayRef<char>(const_cast<char *>(data.data()), data.size());
  }

  /// Return if the data of this blob is mutable.
  bool isMutable() const { return dataIsMutable; }

  /// Return the deleter function of this blob.
  DeleterFn &getDeleter() { return deleter; }
  const DeleterFn &getDeleter() const { return deleter; }

private:
  /// The raw, properly aligned, blob data.
  ArrayRef<char> data;

  /// The alignment of the data.
  size_t dataAlignment = 0;

  /// An optional deleter function used to deallocate the underlying data when
  /// necessary.
  DeleterFn deleter;

  /// Whether the data is mutable.
  bool dataIsMutable;
````
- **EN**: This block groups callable interfaces such as `getDataAlignment`, `getData`, `getDataAs`, `data`, indicating how `AsmState` is queried or updated.
- **CN**: 该代码块聚合了 `getDataAlignment`, `getData`, `getDataAs`, `data` 等可调用接口，展示了如何查询或更新 `AsmState`。

### Lines 183-240
````cpp
  friend class HeapAsmResourceBlob;
};

/// This class provides a simple utility wrapper for creating heap allocated
/// AsmResourceBlobs.
class HeapAsmResourceBlob {
public:
  /// Create a new heap allocated blob with the given size and alignment.
  /// `dataIsMutable` indicates if the allocated data can be mutated. By
  /// default, we treat heap allocated blobs as mutable.
  static AsmResourceBlob allocate(size_t size, size_t align,
                                  bool dataIsMutable = true) {
    return AsmResourceBlob(
        ArrayRef<char>((char *)llvm::allocate_buffer(size, align), size), align,
        llvm::deallocate_buffer, dataIsMutable);
  }
  /// Create a new heap allocated blob and copy the provided data into it.
  static AsmResourceBlob allocateAndCopyWithAlign(ArrayRef<char> data,
                                                  size_t align,
                                                  bool dataIsMutable = true) {
    // This sets the blob to be mutable initially to allow writing
    // (getMutableData) below.
    AsmResourceBlob blob = allocate(data.size(), align, /*dataIsMutable=*/true);
    std::memcpy(blob.getMutableData().data(), data.data(), data.size());
    blob.dataIsMutable = dataIsMutable;
    return blob;
  }
  template <typename T>
  static AsmResourceBlob allocateAndCopyInferAlign(ArrayRef<T> data,
                                                   bool dataIsMutable = true) {
    return allocateAndCopyWithAlign(
        ArrayRef<char>((const char *)data.data(), data.size() * sizeof(T)),
        alignof(T), dataIsMutable);
  }
};
/// This class provides a simple utility wrapper for creating "unmanaged"
/// AsmResourceBlobs. The lifetime of the data provided to these blobs is
/// guaranteed to persist beyond the lifetime of this reference.
class UnmanagedAsmResourceBlob {
public:
  /// Create a new unmanaged resource directly referencing the provided data.
  /// `dataIsMutable` indicates if the allocated data can be mutated. By
  /// default, we treat unmanaged blobs as immutable.
  static AsmResourceBlob
  allocateWithAlign(ArrayRef<char> data, size_t align,
                    AsmResourceBlob::DeleterFn deleter = {},
                    bool dataIsMutable = false) {
    return AsmResourceBlob(data, align, std::move(deleter), dataIsMutable);
  }
  template <typename T>
  static AsmResourceBlob
  allocateInferAlign(ArrayRef<T> data, AsmResourceBlob::DeleterFn deleter = {},
                     bool dataIsMutable = false) {
    return allocateWithAlign(
        ArrayRef<char>((const char *)data.data(), data.size() * sizeof(T)),
        alignof(T), std::move(deleter), dataIsMutable);
  }
};
````
- **EN**: This C++ declaration introduces `HeapAsmResourceBlob` and establishes part of the API surface for `AsmState`. Representative entry points here include `allocate`, `AsmResourceBlob`, `allocate_buffer`, `allocateAndCopyWithAlign`.
- **CN**: 该 C++ 声明引入了 `HeapAsmResourceBlob`，并构成 `AsmState` API 表面的一部分。 这一段可见的代表性接口包括 `allocate`, `AsmResourceBlob`, `allocate_buffer`, `allocateAndCopyWithAlign`。

### Lines 242-287
````cpp
/// This class is used to build resource entries for use by the printer. Each
/// resource entry is represented using a key/value pair. The provided key must
/// be unique within the current context, which allows for a client to provide
/// resource entries without worrying about overlap with other clients.
class AsmResourceBuilder {
public:
  virtual ~AsmResourceBuilder();

  /// Build a resource entry represented by the given bool.
  virtual void buildBool(StringRef key, bool data) = 0;

  /// Build a resource entry represented by the given human-readable string
  /// value.
  virtual void buildString(StringRef key, StringRef data) = 0;

  /// Build an resource entry represented by the given binary blob data.
  virtual void buildBlob(StringRef key, ArrayRef<char> data,
                         uint32_t dataAlignment) = 0;
  /// Build an resource entry represented by the given binary blob data. This is
  /// a useful overload if the data type is known. Note that this does not
  /// support `char` element types to avoid accidentally not providing the
  /// expected alignment of data in situations that treat blobs generically.
  template <typename T>
  std::enable_if_t<!std::is_same<T, char>::value> buildBlob(StringRef key,
                                                            ArrayRef<T> data) {
    buildBlob(
        key, ArrayRef<char>((const char *)data.data(), data.size() * sizeof(T)),
        alignof(T));
  }
  /// Build an resource entry represented by the given resource blob. This is
  /// a useful overload if a blob already exists in-memory.
  void buildBlob(StringRef key, const AsmResourceBlob &blob) {
    buildBlob(key, blob.getData(), blob.getDataAlignment());
  }
};

/// This enum represents the different kinds of resource values.
enum class AsmResourceEntryKind {
  /// A blob of data with an accompanying alignment.
  Blob,
  /// A boolean value.
  Bool,
  /// A string value.
  String,
};
StringRef toString(AsmResourceEntryKind kind);
````
- **EN**: This C++ declaration introduces `AsmResourceBuilder` and establishes part of the API surface for `AsmState`. Representative entry points here include `AsmResourceBuilder`, `buildBool`, `buildString`, `buildBlob`.
- **CN**: 该 C++ 声明引入了 `AsmResourceBuilder`，并构成 `AsmState` API 表面的一部分。 这一段可见的代表性接口包括 `AsmResourceBuilder`, `buildBool`, `buildString`, `buildBlob`。

### Lines 289-329
````cpp
/// This class represents a single parsed resource entry.
class AsmParsedResourceEntry {
public:
  virtual ~AsmParsedResourceEntry();

  /// Return the key of the resource entry.
  virtual StringRef getKey() const = 0;

  /// Emit an error at the location of this entry.
  virtual InFlightDiagnostic emitError() const = 0;

  /// Return the kind of this value.
  virtual AsmResourceEntryKind getKind() const = 0;

  /// Parse the resource entry represented by a boolean. Returns failure if the
  /// entry does not correspond to a bool.
  virtual FailureOr<bool> parseAsBool() const = 0;

  /// Parse the resource entry represented by a human-readable string. Returns
  /// failure if the entry does not correspond to a string.
  virtual FailureOr<std::string> parseAsString() const = 0;

  /// An allocator function used to allocate memory for a blob when required.
  /// The function is provided a size and alignment, and should return an
  /// aligned allocation buffer.
  using BlobAllocatorFn =
      function_ref<AsmResourceBlob(size_t size, size_t align)>;

  /// Parse the resource entry represented by a binary blob. Returns failure if
  /// the entry does not correspond to a blob. If the blob needed to be
  /// allocated, the given allocator function is invoked.
  virtual FailureOr<AsmResourceBlob>
  parseAsBlob(BlobAllocatorFn allocator) const = 0;
  /// Parse the resource entry represented by a binary blob using heap
  /// allocation.
  FailureOr<AsmResourceBlob> parseAsBlob() const {
    return parseAsBlob([](size_t size, size_t align) {
      return HeapAsmResourceBlob::allocate(size, align);
    });
  }
};
````
- **EN**: This C++ declaration introduces `AsmParsedResourceEntry` and establishes part of the API surface for `AsmState`. Representative entry points here include `AsmParsedResourceEntry`, `getKey`, `emitError`, `getKind`.
- **CN**: 该 C++ 声明引入了 `AsmParsedResourceEntry`，并构成 `AsmState` API 表面的一部分。 这一段可见的代表性接口包括 `AsmParsedResourceEntry`, `getKey`, `emitError`, `getKind`。

### Lines 332-379
````cpp
//===----------------------------------------------------------------------===//
// Resource Parser/Printer
//===----------------------------------------------------------------------===//

/// This class represents an instance of a resource parser. This class should be
/// implemented by non-dialect clients that want to inject additional resources
/// into MLIR assembly formats.
class AsmResourceParser {
public:
  /// Create a new parser with the given identifying name. This name uniquely
  /// identifies the entries of this parser, and differentiates them from other
  /// contexts.
  AsmResourceParser(StringRef name) : name(name.str()) {}
  virtual ~AsmResourceParser();

  /// Return the name of this parser.
  StringRef getName() const { return name; }

  /// Parse the given resource entry. Returns failure if the key/data were not
  /// valid, or could otherwise not be processed correctly. Any necessary errors
  /// should be emitted with the provided entry.
  virtual LogicalResult parseResource(AsmParsedResourceEntry &entry) = 0;

  /// Return a resource parser implemented via the given callable, whose form
  /// should match that of `parseResource` above.
  template <typename CallableT>
  static std::unique_ptr<AsmResourceParser> fromCallable(StringRef name,
                                                         CallableT &&parseFn) {
    struct Processor : public AsmResourceParser {
      Processor(StringRef name, CallableT &&parseFn)
          : AsmResourceParser(name), parseFn(std::move(parseFn)) {}
      LogicalResult parseResource(AsmParsedResourceEntry &entry) override {
        return parseFn(entry);
      }

      std::decay_t<CallableT> parseFn;
    };
    return std::make_unique<Processor>(name, std::forward<CallableT>(parseFn));
  }

private:
  std::string name;
};

/// This class represents an instance of a resource printer. This class should
/// be implemented by non-dialect clients that want to inject additional
/// resources into MLIR assembly formats.
class AsmResourcePrinter {
````
- **EN**: This C++ declaration introduces `AsmResourceParser` and establishes part of the API surface for `AsmState`. Representative entry points here include `AsmResourceParser`, `name`, `str`, `getName`.
- **CN**: 该 C++ 声明引入了 `AsmResourceParser`，并构成 `AsmState` API 表面的一部分。 这一段可见的代表性接口包括 `AsmResourceParser`, `name`, `str`, `getName`。

### Lines 380-427
````cpp
public:
  /// Create a new printer with the given identifying name. This name uniquely
  /// identifies the entries of this printer, and differentiates them from
  /// other contexts.
  AsmResourcePrinter(StringRef name) : name(name.str()) {}
  virtual ~AsmResourcePrinter();

  /// Return the name of this printer.
  StringRef getName() const { return name; }

  /// Build any resources to include during printing, utilizing the given
  /// top-level root operation to help determine what information to include.
  /// Provided data should be registered in the form of a key/data pair, to the
  /// given builder.
  virtual void buildResources(Operation *op,
                              AsmResourceBuilder &builder) const = 0;

  /// Return a resource printer implemented via the given callable, whose form
  /// should match that of `buildResources` above.
  template <typename CallableT>
  static std::unique_ptr<AsmResourcePrinter> fromCallable(StringRef name,
                                                          CallableT &&printFn) {
    struct Printer : public AsmResourcePrinter {
      Printer(StringRef name, CallableT &&printFn)
          : AsmResourcePrinter(name), printFn(std::move(printFn)) {}
      void buildResources(Operation *op,
                          AsmResourceBuilder &builder) const override {
        printFn(op, builder);
      }

      std::decay_t<CallableT> printFn;
    };
    return std::make_unique<Printer>(name, std::forward<CallableT>(printFn));
  }

private:
  std::string name;
};

/// A fallback map containing external resources not explicitly handled by
/// another parser/printer.
class FallbackAsmResourceMap {
public:
  /// This class represents an opaque resource.
  struct OpaqueAsmResource {
    OpaqueAsmResource(StringRef key,
                      std::variant<AsmResourceBlob, bool, std::string> value)
        : key(key.str()), value(std::move(value)) {}
````
- **EN**: This C++ declaration introduces `Printer` and establishes part of the API surface for `AsmState`. Representative entry points here include `AsmResourcePrinter`, `name`, `str`, `getName`.
- **CN**: 该 C++ 声明引入了 `Printer`，并构成 `AsmState` API 表面的一部分。 这一段可见的代表性接口包括 `AsmResourcePrinter`, `name`, `str`, `getName`。

### Lines 428-460
````cpp
    /// The key identifying the resource.
    std::string key;
    /// An opaque value for the resource, whose variant values align 1-1 with
    /// the kinds defined in AsmResourceEntryKind.
    std::variant<AsmResourceBlob, bool, std::string> value;
  };

  /// Return a parser than can be used for parsing entries for the given
  /// identifier key.
  AsmResourceParser &getParserFor(StringRef key);

  /// Build a set of resource printers to print the resources within this map.
  std::vector<std::unique_ptr<AsmResourcePrinter>> getPrinters();

private:
  struct ResourceCollection : public AsmResourceParser {
    ResourceCollection(StringRef name) : AsmResourceParser(name) {}

    /// Parse a resource into this collection.
    LogicalResult parseResource(AsmParsedResourceEntry &entry) final;

    /// Build the resources held by this collection.
    void buildResources(Operation *op, AsmResourceBuilder &builder) const;

    /// The set of resources parsed into this collection.
    SmallVector<OpaqueAsmResource> resources;
  };

  /// The set of opaque resources.
  llvm::MapVector<std::string, std::unique_ptr<ResourceCollection>,
                  llvm::StringMap<unsigned>>
      keyToResources;
};
````
- **EN**: This C++ declaration introduces `ResourceCollection` and establishes part of the API surface for `AsmState`. Representative entry points here include `getParserFor`, `getPrinters`, `ResourceCollection`, `AsmResourceParser`.
- **CN**: 该 C++ 声明引入了 `ResourceCollection`，并构成 `AsmState` API 表面的一部分。 这一段可见的代表性接口包括 `getParserFor`, `getPrinters`, `ResourceCollection`, `AsmResourceParser`。

### Lines 463-511
````cpp
//===----------------------------------------------------------------------===//
// ParserConfig
//===----------------------------------------------------------------------===//

/// This class represents a configuration for the MLIR assembly parser. It
/// contains all of the necessary state to parse a MLIR source file.
class ParserConfig {
public:
  /// Construct a parser configuration with the given context.
  /// `verifyAfterParse` indicates if the IR should be verified after parsing.
  /// `fallbackResourceMap` is an optional fallback handler that can be used to
  /// parse external resources not explicitly handled by another parser.
  ParserConfig(MLIRContext *context, bool verifyAfterParse = true,
               FallbackAsmResourceMap *fallbackResourceMap = nullptr)
      : context(context), verifyAfterParse(verifyAfterParse),
        fallbackResourceMap(fallbackResourceMap) {
    assert(context && "expected valid MLIR context");
  }

  /// Return the MLIRContext to be used when parsing.
  MLIRContext *getContext() const { return context; }

  /// Returns if the parser should verify the IR after parsing.
  bool shouldVerifyAfterParse() const { return verifyAfterParse; }

  /// Returns the parsing configurations associated to the bytecode read.
  BytecodeReaderConfig &getBytecodeReaderConfig() const {
    return const_cast<BytecodeReaderConfig &>(bytecodeReaderConfig);
  }

  /// Return the resource parser registered to the given name, or nullptr if no
  /// parser with `name` is registered.
  AsmResourceParser *getResourceParser(StringRef name) const {
    auto it = resourceParsers.find(name);
    if (it != resourceParsers.end())
      return it->second.get();
    if (fallbackResourceMap)
      return &fallbackResourceMap->getParserFor(name);
    return nullptr;
  }

  /// Attach the given resource parser.
  void attachResourceParser(std::unique_ptr<AsmResourceParser> parser) {
    StringRef name = parser->getName();
    auto it = resourceParsers.try_emplace(name, std::move(parser));
    (void)it;
    assert(it.second &&
           "resource parser already registered with the given name");
  }
````
- **EN**: This C++ declaration introduces `ParserConfig` and establishes part of the API surface for `AsmState`. Representative entry points here include `ParserConfig`, `context`, `verifyAfterParse`, `fallbackResourceMap`.
- **CN**: 该 C++ 声明引入了 `ParserConfig`，并构成 `AsmState` API 表面的一部分。 这一段可见的代表性接口包括 `ParserConfig`, `context`, `verifyAfterParse`, `fallbackResourceMap`。

### Lines 512-527
````cpp
  /// Attach the given callable resource parser with the given name.
  template <typename CallableT>
  std::enable_if_t<std::is_convertible<
      CallableT, function_ref<LogicalResult(AsmParsedResourceEntry &)>>::value>
  attachResourceParser(StringRef name, CallableT &&parserFn) {
    attachResourceParser(AsmResourceParser::fromCallable(
        name, std::forward<CallableT>(parserFn)));
  }

private:
  MLIRContext *context;
  bool verifyAfterParse;
  DenseMap<StringRef, std::unique_ptr<AsmResourceParser>> resourceParsers;
  FallbackAsmResourceMap *fallbackResourceMap;
  BytecodeReaderConfig bytecodeReaderConfig;
};
````
- **EN**: This block groups callable interfaces such as `LogicalResult`, `attachResourceParser`, `fromCallable`, indicating how `AsmState` is queried or updated.
- **CN**: 该代码块聚合了 `LogicalResult`, `attachResourceParser`, `fromCallable` 等可调用接口，展示了如何查询或更新 `AsmState`。

### Lines 530-568
````cpp
//===----------------------------------------------------------------------===//
// AsmState
//===----------------------------------------------------------------------===//

/// This class provides management for the lifetime of the state used when
/// printing the IR. It allows for alleviating the cost of recomputing the
/// internal state of the asm printer.
///
/// The IR should not be mutated in-between invocations using this state, and
/// the IR being printed must not be an parent of the IR originally used to
/// initialize this state. This means that if a child operation is provided, a
/// parent operation cannot reuse this state.
class AsmState {
public:
  /// This map represents the raw locations of operations within the output
  /// stream. This maps the original pointer to the operation, to a pair of line
  /// and column in the output stream.
  using LocationMap = DenseMap<Operation *, std::pair<unsigned, unsigned>>;

  /// Initialize the asm state at the level of the given operation. A location
  /// map may optionally be provided to be populated when printing. `map` is an
  /// optional fallback resource map, which when provided will attach resource
  /// printers for the fallback resources within the map.
  AsmState(Operation *op,
           const OpPrintingFlags &printerFlags = OpPrintingFlags(),
           LocationMap *locationMap = nullptr,
           FallbackAsmResourceMap *map = nullptr);
  AsmState(MLIRContext *ctx,
           const OpPrintingFlags &printerFlags = OpPrintingFlags(),
           LocationMap *locationMap = nullptr,
           FallbackAsmResourceMap *map = nullptr);
  ~AsmState();

  /// Get the printer flags.
  const OpPrintingFlags &getPrinterFlags() const;

  /// Return an instance of the internal implementation. Returns nullptr if the
  /// state has not been initialized.
  detail::AsmStateImpl &getImpl() { return *impl; }
````
- **EN**: This C++ declaration introduces `AsmState` and establishes part of the API surface for `AsmState`. Representative entry points here include `AsmState`, `OpPrintingFlags`, `getPrinterFlags`, `getImpl`.
- **CN**: 该 C++ 声明引入了 `AsmState`，并构成 `AsmState` API 表面的一部分。 这一段可见的代表性接口包括 `AsmState`, `OpPrintingFlags`, `getPrinterFlags`, `getImpl`。

### Lines 570-603
````cpp
  //===--------------------------------------------------------------------===//
  // Resources
  //===--------------------------------------------------------------------===//

  /// Attach the given resource printer to the AsmState.
  void attachResourcePrinter(std::unique_ptr<AsmResourcePrinter> printer);

  /// Attach an resource printer, in the form of a callable, to the AsmState.
  template <typename CallableT>
  std::enable_if_t<std::is_convertible<
      CallableT, function_ref<void(Operation *, AsmResourceBuilder &)>>::value>
  attachResourcePrinter(StringRef name, CallableT &&printFn) {
    attachResourcePrinter(AsmResourcePrinter::fromCallable(
        name, std::forward<CallableT>(printFn)));
  }

  /// Attach resource printers to the AsmState for the fallback resources
  /// in the given map.
  void attachFallbackResourcePrinter(FallbackAsmResourceMap &map) {
    for (auto &printer : map.getPrinters())
      attachResourcePrinter(std::move(printer));
  }

  /// Returns a map of dialect resources that were referenced when using this
  /// state to print IR.
  DenseMap<Dialect *, SetVector<AsmDialectResourceHandle>> &
  getDialectResources() const;

private:
  AsmState() = delete;

  /// A pointer to allocated storage for the impl state.
  std::unique_ptr<detail::AsmStateImpl> impl;
};
````
- **EN**: This block groups callable interfaces such as `attachResourcePrinter`, `void`, `fromCallable`, `attachFallbackResourcePrinter`, indicating how `AsmState` is queried or updated.
- **CN**: 该代码块聚合了 `attachResourcePrinter`, `void`, `fromCallable`, `attachFallbackResourcePrinter` 等可调用接口，展示了如何查询或更新 `AsmState`。

### Lines 605-613
````cpp
//===----------------------------------------------------------------------===//
// AsmPrinter CommandLine Options
//===----------------------------------------------------------------------===//

/// Register a set of useful command-line options that can be used to configure
/// various flags within the AsmPrinter.
void registerAsmPrinterCLOptions();

} // namespace mlir
````
- **EN**: This block groups callable interfaces such as `registerAsmPrinterCLOptions`, indicating how `AsmState` is queried or updated.
- **CN**: 该代码块聚合了 `registerAsmPrinterCLOptions` 等可调用接口，展示了如何查询或更新 `AsmState`。

### Lines 615-615
````cpp
#endif // MLIR_IR_ASMSTATE_H_
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: C++ declaration surface for MLIR infrastructure
  **CN**: MLIR 基础设施的 C++ 声明层

## Dependencies / 依赖关系

- mlir/Bytecode/BytecodeReaderConfig.h
- mlir/IR/OperationSupport.h
- mlir/Support/LLVM.h
- llvm/ADT/MapVector.h
- llvm/ADT/StringMap.h

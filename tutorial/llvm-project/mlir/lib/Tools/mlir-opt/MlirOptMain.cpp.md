# MlirOptMain.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Tools/mlir-opt/MlirOptMain.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: This is a utility that runs an optimization pass and prints the result back out. It is designed to support unit testing.
  - **CN**: 实现构建于 MLIR 库之上的可复用命令行工具支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28
```cpp
//===- MlirOptMain.cpp - MLIR Optimizer Driver ----------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This is a utility that runs an optimization pass and prints the result back
// out. It is designed to support unit testing.
//
//===----------------------------------------------------------------------===//

#include "mlir/Tools/mlir-opt/MlirOptMain.h"
#include "mlir/Bytecode/BytecodeWriter.h"
#include "mlir/Debug/CLOptionsSetup.h"
#include "mlir/Debug/Counter.h"
#include "mlir/Dialect/IRDL/IR/IRDL.h"
#include "mlir/Dialect/IRDL/IRDLLoading.h"
#include "mlir/IR/AsmState.h"
#include "mlir/IR/Attributes.h"
#include "mlir/IR/BuiltinOps.h"
#include "mlir/IR/Diagnostics.h"
#include "mlir/IR/Location.h"
#include "mlir/IR/MLIRContext.h"
#include "mlir/IR/Remarks.h"
#include "mlir/Parser/Parser.h"
#include "mlir/Pass/PassManager.h"
```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Tools/mlir-opt/MlirOptMain.h`, `mlir/Bytecode/BytecodeWriter.h`, `mlir/Debug/CLOptionsSetup.h`, `mlir/Debug/Counter.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Tools/mlir-opt/MlirOptMain.h`, `mlir/Bytecode/BytecodeWriter.h`, `mlir/Debug/CLOptionsSetup.h`, `mlir/Debug/Counter.h`。

### Lines 29-49
```cpp
#include "mlir/Pass/PassRegistry.h"
#include "mlir/Remark/RemarkStreamer.h"
#include "mlir/Support/FileUtilities.h"
#include "mlir/Support/Timing.h"
#include "mlir/Support/ToolUtilities.h"
#include "mlir/Tools/ParseUtilities.h"
#include "mlir/Tools/Plugins/DialectPlugin.h"
#include "mlir/Tools/Plugins/PassPlugin.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Remarks/RemarkFormat.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/InitLLVM.h"
#include "llvm/Support/LogicalResult.h"
#include "llvm/Support/ManagedStatic.h"
#include "llvm/Support/Process.h"
#include "llvm/Support/Regex.h"
#include "llvm/Support/SourceMgr.h"
#include "llvm/Support/ThreadPool.h"
#include "llvm/Support/ToolOutputFile.h"

```
- **EN**: Pulls in the declarations needed by this translation unit, including `mlir/Pass/PassRegistry.h`, `mlir/Remark/RemarkStreamer.h`, `mlir/Support/FileUtilities.h`, `mlir/Support/Timing.h`.
- **CN**: 引入该编译单元所需的声明，其中包括 `mlir/Pass/PassRegistry.h`, `mlir/Remark/RemarkStreamer.h`, `mlir/Support/FileUtilities.h`, `mlir/Support/Timing.h`。

### Lines 50-69
```cpp
using namespace mlir;
using namespace llvm;

namespace {
class BytecodeVersionParser : public cl::parser<std::optional<int64_t>> {
public:
  BytecodeVersionParser(cl::Option &o)
      : cl::parser<std::optional<int64_t>>(o) {}

  bool parse(cl::Option &o, StringRef /*argName*/, StringRef arg,
             std::optional<int64_t> &v) {
    long long w;
    if (getAsSignedInteger(arg, 10, w))
      return o.error("Invalid argument '" + arg +
                     "', only integer is supported.");
    v = w;
    return false;
  }
};

```
- **EN**: Introduces declarations for `BytecodeVersionParser`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `BytecodeVersionParser` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 70-83
```cpp
/// This class is intended to manage the handling of command line options for
/// creating a *-opt config. This is a singleton.
struct MlirOptMainConfigCLOptions : public MlirOptMainConfig {
  MlirOptMainConfigCLOptions() {
    // These options are static but all uses ExternalStorage to initialize the
    // members of the parent class. This is unusual but since this class is a
    // singleton it basically attaches command line option to the singleton
    // members.

    static cl::opt<bool, /*ExternalStorage=*/true> allowUnregisteredDialects(
        "allow-unregistered-dialect",
        cl::desc("Allow operation with no registered dialects"),
        cl::location(allowUnregisteredDialectsFlag), cl::init(false));

```
- **EN**: Introduces declarations for `MlirOptMainConfigCLOptions`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `MlirOptMainConfigCLOptions` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 84-101
```cpp
    static cl::opt<bool, /*ExternalStorage=*/true> dumpPassPipeline(
        "dump-pass-pipeline", cl::desc("Print the pipeline that will be run"),
        cl::location(dumpPassPipelineFlag), cl::init(false));

    static cl::opt<bool, /*ExternalStorage=*/true> emitBytecode(
        "emit-bytecode", cl::desc("Emit bytecode when generating output"),
        cl::location(emitBytecodeFlag), cl::init(false));

    static cl::opt<bool, /*ExternalStorage=*/true> elideResourcesFromBytecode(
        "elide-resource-data-from-bytecode",
        cl::desc("Elide resources when generating bytecode"),
        cl::location(elideResourceDataFromBytecodeFlag), cl::init(false));

    static cl::opt<std::string, /*ExternalStorage=*/true> emitBytecodeProducer(
        "emit-bytecode-producer",
        cl::desc("Use specified producer when generating bytecode output"),
        cl::location(emitBytecodeProducerFlag), cl::init(""));

```
- **EN**: Implements logic around `dumpPassPipeline`, `desc`, `location`, `emitBytecode`, and 2 more symbols; this block coordinates pass execution or analysis state; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `dumpPassPipeline`、`desc`、`location`、`emitBytecode` 等另外 2 个符号 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 102-126
```cpp
    static cl::opt<std::optional<int64_t>, /*ExternalStorage=*/true,
                   BytecodeVersionParser>
        bytecodeVersion(
            "emit-bytecode-version",
            cl::desc("Use specified bytecode when generating output"),
            cl::location(emitBytecodeVersion), cl::init(std::nullopt));

    static cl::opt<std::string, /*ExternalStorage=*/true> irdlFile(
        "irdl-file",
        cl::desc("IRDL file to register before processing the input"),
        cl::location(irdlFileFlag), cl::init(""), cl::value_desc("filename"));

    static cl::opt<VerbosityLevel, /*ExternalStorage=*/true>
        diagnosticVerbosityLevel(
            "mlir-diagnostic-verbosity-level",
            cl::desc("Choose level of diagnostic information"),
            cl::location(diagnosticVerbosityLevelFlag),
            cl::init(VerbosityLevel::ErrorsWarningsAndRemarks),
            cl::values(
                clEnumValN(VerbosityLevel::ErrorsOnly, "errors", "Errors only"),
                clEnumValN(VerbosityLevel::ErrorsAndWarnings, "warnings",
                           "Errors and warnings"),
                clEnumValN(VerbosityLevel::ErrorsWarningsAndRemarks, "remarks",
                           "Errors, warnings and remarks")));

```
- **EN**: Implements logic around `bytecodeVersion`, `desc`, `location`, `irdlFile`, and 4 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects.
- **CN**: 围绕 `bytecodeVersion`、`desc`、`location`、`irdlFile` 等另外 4 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象。

### Lines 127-140
```cpp
    static cl::opt<bool, /*ExternalStorage=*/true> disableDiagnosticNotes(
        "mlir-disable-diagnostic-notes", cl::desc("Disable diagnostic notes."),
        cl::location(disableDiagnosticNotesFlag), cl::init(false));

    static cl::opt<bool, /*ExternalStorage=*/true> explicitModule(
        "no-implicit-module",
        cl::desc("Disable implicit addition of a top-level module op during "
                 "parsing"),
        cl::location(useExplicitModuleFlag), cl::init(false));

    static cl::opt<bool, /*ExternalStorage=*/true> listPasses(
        "list-passes", cl::desc("Print the list of registered passes and exit"),
        cl::location(listPassesFlag), cl::init(false));

```
- **EN**: Implements logic around `disableDiagnosticNotes`, `desc`, `location`, `explicitModule`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `disableDiagnosticNotes`、`desc`、`location`、`explicitModule` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 141-162
```cpp
    static cl::opt<bool, /*ExternalStorage=*/true> runReproducer(
        "run-reproducer", cl::desc("Run the pipeline stored in the reproducer"),
        cl::location(runReproducerFlag), cl::init(false));

    static cl::opt<bool, /*ExternalStorage=*/true> showDialects(
        "show-dialects",
        cl::desc("Print the list of registered dialects and exit"),
        cl::location(showDialectsFlag), cl::init(false));

    static cl::opt<std::string, /*ExternalStorage=*/true> splitInputFile{
        "split-input-file",
        llvm::cl::ValueOptional,
        cl::callback([&](const std::string &str) {
          // Implicit value: use default marker if flag was used without value.
          if (str.empty())
            splitInputFile.setValue(kDefaultSplitMarker);
        }),
        cl::desc("Split the input file into chunks using the given or "
                 "default marker and process each chunk independently"),
        cl::location(splitInputFileFlag),
        cl::init("")};

```
- **EN**: Implements logic around `runReproducer`, `desc`, `location`, `showDialects`, and 4 more symbols; this block coordinates pass execution or analysis state; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `runReproducer`、`desc`、`location`、`showDialects` 等另外 4 个符号 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 163-188
```cpp
    static cl::opt<std::string, /*ExternalStorage=*/true> outputSplitMarker(
        "output-split-marker",
        cl::desc("Split marker to use for merging the ouput"),
        cl::location(outputSplitMarkerFlag), cl::init(kDefaultSplitMarker));

    static cl::opt<SourceMgrDiagnosticVerifierHandler::Level,
                   /*ExternalStorage=*/true>
        verifyDiagnostics{
            "verify-diagnostics", llvm::cl::ValueOptional,
            cl::desc("Check that emitted diagnostics match expected-* lines on "
                     "the corresponding line"),
            cl::location(verifyDiagnosticsFlag),
            cl::values(
                clEnumValN(SourceMgrDiagnosticVerifierHandler::Level::All,
                           "all",
                           "Check all diagnostics (expected, unexpected, "
                           "near-misses)"),
                // Implicit value: when passed with no arguments, e.g.
                // `--verify-diagnostics` or `--verify-diagnostics=`.
                clEnumValN(SourceMgrDiagnosticVerifierHandler::Level::All, "",
                           "Check all diagnostics (expected, unexpected, "
                           "near-misses)"),
                clEnumValN(
                    SourceMgrDiagnosticVerifierHandler::Level::OnlyExpected,
                    "only-expected", "Check only expected diagnostics"))};

```
- **EN**: Implements logic around `outputSplitMarker`, `desc`, `location`, `values`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `outputSplitMarker`、`desc`、`location`、`values` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 189-203
```cpp
    static cl::opt<bool, /*ExternalStorage=*/true> verifyPasses(
        "verify-each",
        cl::desc("Run the verifier after each transformation pass"),
        cl::location(verifyPassesFlag), cl::init(true));

    static cl::opt<bool, /*ExternalStorage=*/true> disableVerifyOnParsing(
        "mlir-very-unsafe-disable-verifier-on-parsing",
        cl::desc("Disable the verifier on parsing (very unsafe)"),
        cl::location(disableVerifierOnParsingFlag), cl::init(false));

    static cl::opt<bool, /*ExternalStorage=*/true> verifyRoundtrip(
        "verify-roundtrip",
        cl::desc("Round-trip the IR after parsing and ensure it succeeds"),
        cl::location(verifyRoundtripFlag), cl::init(false));

```
- **EN**: Implements logic around `verifyPasses`, `desc`, `location`, `disableVerifyOnParsing`, and 1 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `verifyPasses`、`desc`、`location`、`disableVerifyOnParsing` 等另外 1 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 204-219
```cpp
    static cl::list<std::string> passPlugins(
        "load-pass-plugin", cl::desc("Load passes from plugin library"));

    static cl::opt<std::string, /*ExternalStorage=*/true>
        generateReproducerFile(
            "mlir-generate-reproducer",
            llvm::cl::desc(
                "Generate an mlir reproducer at the provided filename"
                " (no crash required)"),
            cl::location(generateReproducerFileFlag), cl::init(""),
            cl::value_desc("filename"));

    static cl::OptionCategory remarkCategory(
        "Remark Options",
        "Filter remarks by regular expression (llvm::Regex syntax).");

```
- **EN**: Implements logic around `passPlugins`, `desc`, `generateReproducerFile`, `location`, and 3 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `passPlugins`、`desc`、`generateReproducerFile`、`location` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 220-234
```cpp
    static llvm::cl::opt<RemarkFormat, /*ExternalStorage=*/true> remarkFormat{
        "remark-format",
        llvm::cl::desc("Specify the format for remark output."),
        cl::location(remarkFormatFlag),
        llvm::cl::value_desc("format"),
        llvm::cl::init(RemarkFormat::REMARK_FORMAT_STDOUT),
        llvm::cl::values(clEnumValN(RemarkFormat::REMARK_FORMAT_STDOUT,
                                    "emitRemark",
                                    "Print as emitRemark to command-line"),
                         clEnumValN(RemarkFormat::REMARK_FORMAT_YAML, "yaml",
                                    "Print yaml file"),
                         clEnumValN(RemarkFormat::REMARK_FORMAT_BITSTREAM,
                                    "bitstream", "Print bitstream file")),
        llvm::cl::cat(remarkCategory)};

```
- **EN**: Implements logic around `desc`, `location`, `value_desc`, `init`, and 3 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `desc`、`location`、`value_desc`、`init` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 235-252
```cpp
    static llvm::cl::opt<RemarkPolicy, /*ExternalStorage=*/true> remarkPolicy{
        "remark-policy",
        llvm::cl::desc("Specify the policy for remark output."),
        cl::location(remarkPolicyFlag),
        llvm::cl::value_desc("format"),
        llvm::cl::init(RemarkPolicy::REMARK_POLICY_ALL),
        llvm::cl::values(clEnumValN(RemarkPolicy::REMARK_POLICY_ALL, "all",
                                    "Print all remarks"),
                         clEnumValN(RemarkPolicy::REMARK_POLICY_FINAL, "final",
                                    "Print final remarks")),
        llvm::cl::cat(remarkCategory)};

    static cl::opt<std::string, /*ExternalStorage=*/true> remarksAll(
        "remarks-filter",
        cl::desc("Show all remarks: passed, missed, failed, analysis"),
        cl::location(remarksAllFilterFlag), cl::init(""),
        cl::cat(remarkCategory));

```
- **EN**: Implements logic around `desc`, `location`, `value_desc`, `init`, and 4 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `desc`、`location`、`value_desc`、`init` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 253-270
```cpp
    static cl::opt<std::string, /*ExternalStorage=*/true> remarksFile(
        "remarks-output-file",
        cl::desc(
            "Output file for yaml and bitstream remark formats. Default is "
            "mlir-remarks.yaml or mlir-remarks.bitstream"),
        cl::location(remarksOutputFileFlag), cl::init(""),
        cl::cat(remarkCategory));

    static cl::opt<std::string, /*ExternalStorage=*/true> remarksPassed(
        "remarks-filter-passed", cl::desc("Show passed remarks"),
        cl::location(remarksPassedFilterFlag), cl::init(""),
        cl::cat(remarkCategory));

    static cl::opt<std::string, /*ExternalStorage=*/true> remarksFailed(
        "remarks-filter-failed", cl::desc("Show failed remarks"),
        cl::location(remarksFailedFilterFlag), cl::init(""),
        cl::cat(remarkCategory));

```
- **EN**: Implements logic around `remarksFile`, `desc`, `location`, `cat`, and 2 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `remarksFile`、`desc`、`location`、`cat` 等另外 2 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 271-291
```cpp
    static cl::opt<std::string, /*ExternalStorage=*/true> remarksMissed(
        "remarks-filter-missed", cl::desc("Show missed remarks"),
        cl::location(remarksMissedFilterFlag), cl::init(""),
        cl::cat(remarkCategory));

    static cl::opt<std::string, /*ExternalStorage=*/true> remarksAnalyse(
        "remarks-filter-analyse", cl::desc("Show analysis remarks"),
        cl::location(remarksAnalyseFilterFlag), cl::init(""),
        cl::cat(remarkCategory));

    /// Set the callback to load a pass plugin.
    passPlugins.setCallback([&](const std::string &pluginPath) {
      auto plugin = PassPlugin::load(pluginPath);
      if (!plugin) {
        errs() << "Failed to load passes from '" << pluginPath
               << "'. Request ignored.\n";
        return;
      }
      plugin.get().registerPassRegistryCallbacks();
    });

```
- **EN**: Implements logic around `remarksMissed`, `desc`, `location`, `cat`, and 5 more symbols; this block manipulates core MLIR IR objects.
- **CN**: 围绕 `remarksMissed`、`desc`、`location`、`cat` 等另外 5 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象。

### Lines 292-307
```cpp
    static cl::list<std::string> dialectPlugins(
        "load-dialect-plugin", cl::desc("Load dialects from plugin library"));
    this->dialectPlugins = std::addressof(dialectPlugins);

    static PassPipelineCLParser passPipeline("", "Compiler passes to run", "p");
    setPassPipelineParser(passPipeline);
  }

  /// Set the callback to load a dialect plugin.
  void setDialectPluginsCallback(DialectRegistry &registry);

  /// Pointer to static dialectPlugins variable in constructor, needed by
  /// setDialectPluginsCallback(DialectRegistry&).
  cl::list<std::string> *dialectPlugins = nullptr;
};

```
- **EN**: Implements logic around `dialectPlugins`, `desc`, `addressof`, `passPipeline`, and 2 more symbols.
- **CN**: 围绕 `dialectPlugins`、`desc`、`addressof`、`passPipeline` 等另外 2 个符号 实现具体逻辑。

### Lines 308-335
```cpp
/// A scoped diagnostic handler that suppresses certain diagnostics based on
/// the verbosity level and whether the diagnostic is a note.
class DiagnosticFilter : public ScopedDiagnosticHandler {
public:
  DiagnosticFilter(MLIRContext *ctx, VerbosityLevel verbosityLevel,
                   bool showNotes = true)
      : ScopedDiagnosticHandler(ctx) {
    setHandler([verbosityLevel, showNotes](Diagnostic &diag) {
      auto severity = diag.getSeverity();
      switch (severity) {
      case mlir::DiagnosticSeverity::Error:
        // failure indicates that the error is not handled by the filter and
        // goes through to the default handler. Therefore, the error can be
        // successfully printed.
        return failure();
      case mlir::DiagnosticSeverity::Warning:
        if (verbosityLevel == VerbosityLevel::ErrorsOnly)
          return success();
        else
          return failure();
      case mlir::DiagnosticSeverity::Remark:
        if (verbosityLevel == VerbosityLevel::ErrorsOnly ||
            verbosityLevel == VerbosityLevel::ErrorsAndWarnings)
          return success();
        else
          return failure();
      case mlir::DiagnosticSeverity::Note:
        if (showNotes)
```
- **EN**: Introduces declarations for `DiagnosticFilter`, establishing the types, namespaces, or records used later in the file.
- **CN**: 引入 `DiagnosticFilter` 等声明，建立本文件后续使用的类型、命名空间或记录。

### Lines 336-352
```cpp
          return failure();
        else
          return success();
      }
      llvm_unreachable("Unknown diagnostic severity");
    });
  }
};
} // namespace

ManagedStatic<MlirOptMainConfigCLOptions> clOptionsConfig;

void MlirOptMainConfig::registerCLOptions(DialectRegistry &registry) {
  clOptionsConfig->setDialectPluginsCallback(registry);
  tracing::DebugConfig::registerCLOptions();
}

```
- **EN**: Implements logic around `failure`, `success`, `llvm_unreachable`, `registerCLOptions`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `failure`、`success`、`llvm_unreachable`、`registerCLOptions` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 353-368
```cpp
MlirOptMainConfig MlirOptMainConfig::createFromCLOptions() {
  clOptionsConfig->setDebugConfig(tracing::DebugConfig::createFromCLOptions());
  return *clOptionsConfig;
}

MlirOptMainConfig &MlirOptMainConfig::setPassPipelineParser(
    const PassPipelineCLParser &passPipeline) {
  passPipelineCallback = [&](PassManager &pm) {
    auto errorHandler = [&](const Twine &msg) {
      emitError(UnknownLoc::get(pm.getContext())) << msg;
      return failure();
    };
    if (failed(passPipeline.addToPipeline(pm, errorHandler)))
      return failure();
    if (this->shouldDumpPassPipeline()) {

```
- **EN**: Implements logic around `createFromCLOptions`, `setDebugConfig`, `setPassPipelineParser`, `emitError`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; coordinates pass execution or analysis state.
- **CN**: 围绕 `createFromCLOptions`、`setDebugConfig`、`setPassPipelineParser`、`emitError` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并协调 Pass 执行或分析状态。

### Lines 369-389
```cpp
      pm.dump();
      llvm::errs() << "\n";
    }
    return success();
  };
  return *this;
}

void MlirOptMainConfigCLOptions::setDialectPluginsCallback(
    DialectRegistry &registry) {
  dialectPlugins->setCallback([&](const std::string &pluginPath) {
    auto plugin = DialectPlugin::load(pluginPath);
    if (!plugin) {
      errs() << "Failed to load dialect plugin from '" << pluginPath
             << "'. Request ignored.\n";
      return;
    };
    plugin.get().registerDialectRegistryCallbacks(registry);
  });
}

```
- **EN**: Implements logic around `dump`, `errs`, `success`, `setDialectPluginsCallback`, and 3 more symbols.
- **CN**: 围绕 `dump`、`errs`、`success`、`setDialectPluginsCallback` 等另外 3 个符号 实现具体逻辑。

### Lines 390-407
```cpp
LogicalResult loadIRDLDialects(StringRef irdlFile, MLIRContext &ctx) {
  DialectRegistry registry;
  registry.insert<irdl::IRDLDialect>();
  ctx.appendDialectRegistry(registry);

  // Set up the input file.
  std::string errorMessage;
  std::unique_ptr<MemoryBuffer> file = openInputFile(irdlFile, &errorMessage);
  if (!file) {
    emitError(UnknownLoc::get(&ctx)) << errorMessage;
    return failure();
  }

  // Give the buffer to the source manager.
  // This will be picked up by the parser.
  SourceMgr sourceMgr;
  sourceMgr.AddNewSourceBuffer(std::move(file), SMLoc());

```
- **EN**: Implements logic around `loadIRDLDialects`, `IRDLDialect>`, `appendDialectRegistry`, `openInputFile`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations.
- **CN**: 围绕 `loadIRDLDialects`、`IRDLDialect>`、`appendDialectRegistry`、`openInputFile` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示。

### Lines 408-434
```cpp
  SourceMgrDiagnosticHandler sourceMgrHandler(sourceMgr, &ctx);

  // Parse the input file.
  OwningOpRef<ModuleOp> module(parseSourceFile<ModuleOp>(sourceMgr, &ctx));
  if (!module)
    return failure();

  // Load IRDL dialects.
  return irdl::loadDialects(module.get());
}

// Return success if the module can correctly round-trip. This intended to test
// that the custom printers/parsers are complete.
static LogicalResult doVerifyRoundTrip(Operation *op,
                                       const MlirOptMainConfig &config,
                                       bool useBytecode) {
  // We use a new context to avoid resource handle renaming issue in the diff.
  MLIRContext roundtripContext;
  OwningOpRef<Operation *> roundtripModule;
  roundtripContext.appendDialectRegistry(
      op->getContext()->getDialectRegistry());
  if (op->getContext()->allowsUnregisteredDialects())
    roundtripContext.allowUnregisteredDialects();
  StringRef irdlFile = config.getIrdlFile();
  if (!irdlFile.empty() && failed(loadIRDLDialects(irdlFile, roundtripContext)))
    return failure();

```
- **EN**: Implements logic around `sourceMgrHandler`, `module`, `failure`, `loadDialects`, and 6 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `sourceMgrHandler`、`module`、`failure`、`loadDialects` 等另外 6 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 435-461
```cpp
  std::string testType = (useBytecode) ? "bytecode" : "textual";
  // Print a first time with custom format (or bytecode) and parse it back to
  // the roundtripModule.
  {
    std::string buffer;
    llvm::raw_string_ostream ostream(buffer);
    if (useBytecode) {
      if (failed(writeBytecodeToFile(op, ostream))) {
        op->emitOpError()
            << "failed to write bytecode, cannot verify round-trip.\n";
        return failure();
      }
    } else {
      op->print(ostream,
                OpPrintingFlags().printGenericOpForm().enableDebugInfo());
    }
    FallbackAsmResourceMap fallbackResourceMap;
    ParserConfig parseConfig(&roundtripContext, config.shouldVerifyOnParsing(),
                             &fallbackResourceMap);
    roundtripModule = parseSourceString<Operation *>(buffer, parseConfig);
    if (!roundtripModule) {
      op->emitOpError() << "failed to parse " << testType
                        << " content back, cannot verify round-trip.\n";
      return failure();
    }
  }

```
- **EN**: Implements logic around `ostream`, `failed`, `emitOpError`, `failure`, and 3 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `ostream`、`failed`、`emitOpError`、`failure` 等另外 3 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 462-483
```cpp
  // Print in the generic form for the reference module and the round-tripped
  // one and compare the outputs.
  std::string reference, roundtrip;
  {
    llvm::raw_string_ostream ostreamref(reference);
    op->print(ostreamref,
              OpPrintingFlags().printGenericOpForm().enableDebugInfo());
    llvm::raw_string_ostream ostreamrndtrip(roundtrip);
    roundtripModule.get()->print(
        ostreamrndtrip,
        OpPrintingFlags().printGenericOpForm().enableDebugInfo());
  }
  if (reference != roundtrip) {
    // TODO implement a diff.
    return op->emitOpError()
           << testType
           << " roundTrip testing roundtripped module differs "
              "from reference:\n<<<<<<Reference\n"
           << reference << "\n=====\n"
           << roundtrip << "\n>>>>>roundtripped\n";
  }

```
- **EN**: Implements logic around `ostreamref`, `print`, `OpPrintingFlags`, `ostreamrndtrip`, and 2 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `ostreamref`、`print`、`OpPrintingFlags`、`ostreamrndtrip` 等另外 2 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 484-497
```cpp
  return success();
}

static LogicalResult doVerifyRoundTrip(Operation *op,
                                       const MlirOptMainConfig &config) {
  auto txtStatus = doVerifyRoundTrip(op, config, /*useBytecode=*/false);
  auto bcStatus = doVerifyRoundTrip(op, config, /*useBytecode=*/true);
  return success(succeeded(txtStatus) && succeeded(bcStatus));
}

/// Perform the actions on the input file indicated by the command line flags
/// within the specified context.
///
/// This typically parses the main source file, runs zero or more optimization
```
- **EN**: Implements logic around `success`, `doVerifyRoundTrip`; this block makes success/failure or diagnostics explicit through MLIR result utilities; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `success`、`doVerifyRoundTrip` 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 498-512
```cpp
/// passes, then prints the output.
///
static LogicalResult
performActions(raw_ostream &os,
               const std::shared_ptr<llvm::SourceMgr> &sourceMgr,
               MLIRContext *context, const MlirOptMainConfig &config) {
  DefaultTimingManager tm;
  applyDefaultTimingManagerCLOptions(tm);
  TimingScope timing = tm.getRootScope();

  // Disable multi-threading when parsing the input file. This removes the
  // unnecessary/costly context synchronization when parsing.
  bool wasThreadingEnabled = context->isMultithreadingEnabled();
  context->disableMultithreading();

```
- **EN**: Implements logic around `performActions`, `applyDefaultTimingManagerCLOptions`, `getRootScope`, `isMultithreadingEnabled`, and 1 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations.
- **CN**: 围绕 `performActions`、`applyDefaultTimingManagerCLOptions`、`getRootScope`、`isMultithreadingEnabled` 等另外 1 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示。

### Lines 513-531
```cpp
  // Prepare the parser config, and attach any useful/necessary resource
  // handlers. Unhandled external resources are treated as passthrough, i.e.
  // they are not processed and will be emitted directly to the output
  // untouched.
  PassReproducerOptions reproOptions;
  FallbackAsmResourceMap fallbackResourceMap;
  ParserConfig parseConfig(context, config.shouldVerifyOnParsing(),
                           &fallbackResourceMap);
  if (config.shouldRunReproducer())
    reproOptions.attachResourceParser(parseConfig);

  // Parse the input file and reset the context threading state.
  TimingScope parserTiming = timing.nest("Parser");
  OwningOpRef<Operation *> op = parseSourceFileForTool(
      sourceMgr, parseConfig, !config.shouldUseExplicitModule());
  parserTiming.stop();
  if (!op)
    return failure();

```
- **EN**: Implements logic around `parseConfig`, `shouldRunReproducer`, `attachResourceParser`, `nest`, and 4 more symbols; this block manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `parseConfig`、`shouldRunReproducer`、`attachResourceParser`、`nest` 等另外 4 个符号 实现具体逻辑；该代码块操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 532-552
```cpp
  // Perform round-trip verification if requested
  if (config.shouldVerifyRoundtrip() &&
      failed(doVerifyRoundTrip(op.get(), config)))
    return failure();

  context->enableMultithreading(wasThreadingEnabled);
  // Set the remark categories and policy.
  remark::RemarkCategories cats{
      config.getRemarksAllFilter(), config.getRemarksPassedFilter(),
      config.getRemarksMissedFilter(), config.getRemarksAnalyseFilter(),
      config.getRemarksFailedFilter()};

  mlir::MLIRContext &ctx = *context;
  // Helper to create the appropriate policy based on configuration
  auto createPolicy = [&config]()
      -> std::unique_ptr<mlir::remark::detail::RemarkEmittingPolicyBase> {
    if (config.getRemarkPolicy() == RemarkPolicy::REMARK_POLICY_ALL)
      return std::make_unique<mlir::remark::RemarkEmittingPolicyAll>();
    if (config.getRemarkPolicy() == RemarkPolicy::REMARK_POLICY_FINAL)
      return std::make_unique<mlir::remark::RemarkEmittingPolicyFinal>();

```
- **EN**: Implements logic around `shouldVerifyRoundtrip`, `failed`, `failure`, `enableMultithreading`, and 6 more symbols.
- **CN**: 围绕 `shouldVerifyRoundtrip`、`failed`、`failure`、`enableMultithreading` 等另外 6 个符号 实现具体逻辑。

### Lines 553-572
```cpp
    llvm_unreachable("Invalid remark policy");
  };

  switch (config.getRemarkFormat()) {
  case RemarkFormat::REMARK_FORMAT_STDOUT:
    if (failed(mlir::remark::enableOptimizationRemarks(
            ctx, nullptr, createPolicy(), cats, true /*printAsEmitRemarks*/)))
      return failure();
    break;

  case RemarkFormat::REMARK_FORMAT_YAML: {
    std::string file = config.getRemarksOutputFile().empty()
                           ? "mlir-remarks.yaml"
                           : config.getRemarksOutputFile();
    if (failed(mlir::remark::enableOptimizationRemarksWithLLVMStreamer(
            ctx, file, llvm::remarks::Format::YAML, createPolicy(), cats)))
      return failure();
    break;
  }

```
- **EN**: Implements logic around `llvm_unreachable`, `getRemarkFormat`, `failed`, `createPolicy`, and 2 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `llvm_unreachable`、`getRemarkFormat`、`failed`、`createPolicy` 等另外 2 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 573-594
```cpp
  case RemarkFormat::REMARK_FORMAT_BITSTREAM: {
    std::string file = config.getRemarksOutputFile().empty()
                           ? "mlir-remarks.bitstream"
                           : config.getRemarksOutputFile();
    if (failed(mlir::remark::enableOptimizationRemarksWithLLVMStreamer(
            ctx, file, llvm::remarks::Format::Bitstream, createPolicy(), cats)))
      return failure();
    break;
  }
  }

  // Prepare the pass manager, applying command-line and reproducer options.
  PassManager pm(op.get()->getName(), PassManager::Nesting::Implicit);
  pm.enableVerifier(config.shouldVerifyPasses());
  if (failed(applyPassManagerCLOptions(pm)))
    return failure();
  pm.enableTiming(timing);
  if (config.shouldRunReproducer() && failed(reproOptions.apply(pm)))
    return failure();
  if (failed(config.setupPassPipeline(pm)))
    return failure();

```
- **EN**: Implements logic around `getRemarksOutputFile`, `failed`, `createPolicy`, `failure`, and 4 more symbols; this block coordinates pass execution or analysis state.
- **CN**: 围绕 `getRemarksOutputFile`、`failed`、`createPolicy`、`failure` 等另外 4 个符号 实现具体逻辑；该代码块协调 Pass 执行或分析状态。

### Lines 595-620
```cpp
  // Run the pipeline.
  if (failed(pm.run(*op)))
    return failure();

  // Generate reproducers if requested
  if (!config.getReproducerFilename().empty()) {
    StringRef anchorName = pm.getOpAnchorName();
    const auto &passes = pm.getPasses();
    makeReproducer(anchorName, passes, op.get(),
                   config.getReproducerFilename());
  }

  // Print the output.
  TimingScope outputTiming = timing.nest("Output");
  if (config.shouldEmitBytecode()) {
    std::optional<StringRef> producer = config.bytecodeProducerToEmit();
    BytecodeWriterConfig writerConfig =
        producer ? BytecodeWriterConfig(fallbackResourceMap, producer.value())
                 : BytecodeWriterConfig(fallbackResourceMap);
    if (auto v = config.bytecodeVersionToEmit())
      writerConfig.setDesiredBytecodeVersion(*v);
    if (config.shouldElideResourceDataFromBytecode())
      writerConfig.setElideResourceDataFlag();
    return writeBytecodeToFile(op.get(), os, writerConfig);
  }

```
- **EN**: Implements logic around `failed`, `failure`, `getReproducerFilename`, `getOpAnchorName`, and 11 more symbols; this block coordinates pass execution or analysis state; manipulates core MLIR IR objects; parses or prints textual MLIR representations.
- **CN**: 围绕 `failed`、`failure`、`getReproducerFilename`、`getOpAnchorName` 等另外 11 个符号 实现具体逻辑；该代码块协调 Pass 执行或分析状态，并操作 MLIR 核心 IR 对象，并解析或打印文本形式的 MLIR 表示。

### Lines 621-637
```cpp
  if (config.bytecodeVersionToEmit().has_value())
    return emitError(UnknownLoc::get(pm.getContext()))
           << "bytecode version while not emitting bytecode";

  // Don't re-run the verifier if we already ran the verifier at the end of the
  // pass pipeline.
  AsmState asmState(op.get(),
                    OpPrintingFlags().assumeVerified(
                        config.shouldVerifyPasses() && !pm.empty()),
                    /*locationMap=*/nullptr, &fallbackResourceMap);
  os << OpWithState(op.get(), asmState) << '\n';

  // This is required if the remark policy is final. Otherwise, the remarks are
  // not emitted.
  if (remark::detail::RemarkEngine *engine = ctx.getRemarkEngine())
    engine->getRemarkEmittingPolicy()->finalize();

```
- **EN**: Implements logic around `bytecodeVersionToEmit`, `emitError`, `asmState`, `OpPrintingFlags`, and 4 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; coordinates pass execution or analysis state.
- **CN**: 围绕 `bytecodeVersionToEmit`、`emitError`、`asmState`、`OpPrintingFlags` 等另外 4 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并协调 Pass 执行或分析状态。

### Lines 638-658
```cpp
  return success();
}

/// Parses the memory buffer.  If successfully, run a series of passes against
/// it and print the result.
static LogicalResult
processBuffer(raw_ostream &os, std::unique_ptr<MemoryBuffer> ownedBuffer,
              llvm::MemoryBufferRef sourceBuffer,
              const MlirOptMainConfig &config, DialectRegistry &registry,
              SourceMgrDiagnosticVerifierHandler *verifyHandler,
              llvm::ThreadPoolInterface *threadPool) {
  // Tell sourceMgr about this buffer, which is what the parser will pick up.
  auto sourceMgr = std::make_shared<SourceMgr>();
  // Add the original buffer to the source manager to use for determining
  // locations.
  sourceMgr->AddNewSourceBuffer(
      llvm::MemoryBuffer::getMemBuffer(sourceBuffer,
                                       /*RequiresNullTerminator=*/false),
      SMLoc());
  sourceMgr->AddNewSourceBuffer(std::move(ownedBuffer), SMLoc());

```
- **EN**: Implements logic around `success`, `processBuffer`, `make_shared`, `AddNewSourceBuffer`, and 2 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations.
- **CN**: 围绕 `success`、`processBuffer`、`make_shared`、`AddNewSourceBuffer` 等另外 2 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示。

### Lines 659-675
```cpp
  // Create a context just for the current buffer. Disable threading on
  // creation since we'll inject the thread-pool separately.
  MLIRContext context(registry, MLIRContext::Threading::DISABLED);
  if (threadPool)
    context.setThreadPool(*threadPool);
  if (verifyHandler)
    verifyHandler->registerInContext(&context);

  StringRef irdlFile = config.getIrdlFile();
  if (!irdlFile.empty() && failed(loadIRDLDialects(irdlFile, context)))
    return failure();

  // Parse the input file.
  context.allowUnregisteredDialects(config.shouldAllowUnregisteredDialects());
  if (config.shouldVerifyDiagnostics())
    context.printOpOnDiagnostic(false);

```
- **EN**: Implements logic around `context`, `setThreadPool`, `registerInContext`, `getIrdlFile`, and 5 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `context`、`setThreadPool`、`registerInContext`、`getIrdlFile` 等另外 5 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 676-693
```cpp
  tracing::InstallDebugHandler installDebugHandler(context,
                                                   config.getDebugConfig());

  // If we are in verify diagnostics mode then we have a lot of work to do,
  // otherwise just perform the actions without worrying about it.
  if (!config.shouldVerifyDiagnostics()) {
    SourceMgrDiagnosticHandler sourceMgrHandler(*sourceMgr, &context);
    DiagnosticFilter diagnosticFilter(&context,
                                      config.getDiagnosticVerbosityLevel(),
                                      config.shouldShowNotes());
    return performActions(os, sourceMgr, &context, config);
  }

  // Do any processing requested by command line flags.  We don't care whether
  // these actions succeed or fail, we only care what diagnostics they produce
  // and whether they match our expectations.
  (void)performActions(os, sourceMgr, &context, config);

```
- **EN**: Implements logic around `installDebugHandler`, `getDebugConfig`, `shouldVerifyDiagnostics`, `sourceMgrHandler`, and 4 more symbols.
- **CN**: 围绕 `installDebugHandler`、`getDebugConfig`、`shouldVerifyDiagnostics`、`sourceMgrHandler` 等另外 4 个符号 实现具体逻辑。

### Lines 694-715
```cpp
  return success();
}

std::string mlir::registerCLIOptions(llvm::StringRef toolName,
                                     DialectRegistry &registry) {
  MlirOptMainConfig::registerCLOptions(registry);
  registerAsmPrinterCLOptions();
  registerMLIRContextCLOptions();
  registerPassManagerCLOptions();
  registerDefaultTimingManagerCLOptions();
  tracing::DebugCounter::registerCLOptions();

  // Build the list of dialects as a header for the --help message.
  std::string helpHeader = (toolName + "\nAvailable Dialects: ").str();
  {
    llvm::raw_string_ostream os(helpHeader);
    interleaveComma(registry.getRegisteredDialectNames(), os,
                    [&](auto name) { os << name; });
  }
  return helpHeader;
}

```
- **EN**: Implements logic around `success`, `registerCLIOptions`, `registerCLOptions`, `registerAsmPrinterCLOptions`, and 6 more symbols.
- **CN**: 围绕 `success`、`registerCLIOptions`、`registerCLOptions`、`registerAsmPrinterCLOptions` 等另外 6 个符号 实现具体逻辑。

### Lines 716-735
```cpp
std::pair<std::string, std::string>
mlir::parseCLIOptions(int argc, char **argv, llvm::StringRef helpHeader) {
  static cl::opt<std::string> inputFilename(
      cl::Positional, cl::desc("<input file>"), cl::init("-"));

  static cl::opt<std::string> outputFilename("o", cl::desc("Output filename"),
                                             cl::value_desc("filename"),
                                             cl::init("-"));
  cl::ParseCommandLineOptions(argc, argv, helpHeader);
  return std::make_pair(inputFilename.getValue(), outputFilename.getValue());
}

std::pair<std::string, std::string>
mlir::registerAndParseCLIOptions(int argc, char **argv,
                                 llvm::StringRef toolName,
                                 DialectRegistry &registry) {
  auto helpHeader = registerCLIOptions(toolName, registry);
  return parseCLIOptions(argc, argv, helpHeader);
}

```
- **EN**: Implements logic around `parseCLIOptions`, `inputFilename`, `desc`, `outputFilename`, and 6 more symbols; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `parseCLIOptions`、`inputFilename`、`desc`、`outputFilename` 等另外 6 个符号 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 736-754
```cpp
static LogicalResult printRegisteredDialects(DialectRegistry &registry) {
  llvm::outs() << "Available Dialects: ";
  interleave(registry.getRegisteredDialectNames(), llvm::outs(), ",");
  llvm::outs() << "\n";
  return success();
}

static LogicalResult printRegisteredPassesAndReturn() {
  mlir::printRegisteredPasses();
  return success();
}

LogicalResult mlir::MlirOptMain(llvm::raw_ostream &outputStream,
                                std::unique_ptr<llvm::MemoryBuffer> buffer,
                                DialectRegistry &registry,
                                const MlirOptMainConfig &config) {
  if (config.shouldShowDialects())
    return printRegisteredDialects(registry);

```
- **EN**: Implements logic around `printRegisteredDialects`, `outs`, `interleave`, `success`, and 4 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations.
- **CN**: 围绕 `printRegisteredDialects`、`outs`、`interleave`、`success` 等另外 4 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示。

### Lines 755-771
```cpp
  if (config.shouldListPasses())
    return printRegisteredPassesAndReturn();

  // The split-input-file mode is a very specific mode that slices the file
  // up into small pieces and checks each independently.
  // We use an explicit threadpool to avoid creating and joining/destroying
  // threads for each of the split.
  ThreadPoolInterface *threadPool = nullptr;

  // Create a temporary context for the sake of checking if
  // --mlir-disable-threading was passed on the command line.
  // We use the thread-pool this context is creating, and avoid
  // creating any thread when disabled.
  MLIRContext threadPoolCtx;
  if (threadPoolCtx.isMultithreadingEnabled())
    threadPool = &threadPoolCtx.getThreadPool();

```
- **EN**: Implements logic around `shouldListPasses`, `printRegisteredPassesAndReturn`, `isMultithreadingEnabled`, `getThreadPool`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `shouldListPasses`、`printRegisteredPassesAndReturn`、`isMultithreadingEnabled`、`getThreadPool` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 772-798
```cpp
  SourceMgr sourceMgr;
  sourceMgr.AddNewSourceBuffer(
      llvm::MemoryBuffer::getMemBuffer(buffer->getMemBufferRef(),
                                       /*RequiresNullTerminator=*/false),
      SMLoc());
  // Note: this creates a verifier handler independent of the the flag set, as
  // internally if the flag is not set, a new scoped diagnostic handler is
  // created which would intercept the diagnostics and verify them.
  SourceMgrDiagnosticVerifierHandler sourceMgrHandler(
      sourceMgr, &threadPoolCtx, config.verifyDiagnosticsLevel());
  auto chunkFn = [&](std::unique_ptr<MemoryBuffer> chunkBuffer,
                     llvm::MemoryBufferRef sourceBuffer, raw_ostream &os) {
    return processBuffer(
        os, std::move(chunkBuffer), sourceBuffer, config, registry,
        config.shouldVerifyDiagnostics() ? &sourceMgrHandler : nullptr,
        threadPool);
  };
  LogicalResult status = splitAndProcessBuffer(
      llvm::MemoryBuffer::getMemBuffer(buffer->getMemBufferRef(),
                                       /*RequiresNullTerminator=*/false),
      chunkFn, outputStream, config.inputSplitMarker(),
      config.outputSplitMarker());
  if (config.shouldVerifyDiagnostics() && failed(sourceMgrHandler.verify()))
    status = failure();
  return status;
}

```
- **EN**: Implements logic around `AddNewSourceBuffer`, `getMemBuffer`, `SMLoc`, `sourceMgrHandler`, and 8 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `AddNewSourceBuffer`、`getMemBuffer`、`SMLoc`、`sourceMgrHandler` 等另外 8 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 799-813
```cpp
LogicalResult mlir::MlirOptMain(int argc, char **argv,
                                llvm::StringRef inputFilename,
                                llvm::StringRef outputFilename,
                                DialectRegistry &registry) {

  InitLLVM y(argc, argv);

  MlirOptMainConfig config = MlirOptMainConfig::createFromCLOptions();

  if (config.shouldShowDialects())
    return printRegisteredDialects(registry);

  if (config.shouldListPasses())
    return printRegisteredPassesAndReturn();

```
- **EN**: Implements logic around `MlirOptMain`, `y`, `createFromCLOptions`, `shouldShowDialects`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities; parses or prints textual MLIR representations.
- **CN**: 围绕 `MlirOptMain`、`y`、`createFromCLOptions`、`shouldShowDialects` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断，并解析或打印文本形式的 MLIR 表示。

### Lines 814-829
```cpp
  // When reading from stdin and the input is a tty, it is often a user
  // mistake and the process "appears to be stuck". Print a message to let the
  // user know about it!
  if (inputFilename == "-" &&
      sys::Process::FileDescriptorIsDisplayed(fileno(stdin)))
    llvm::errs() << "(processing input from stdin now, hit ctrl-c/ctrl-d to "
                    "interrupt)\n";

  // Set up the input file.
  std::string errorMessage;
  auto file = openInputFile(inputFilename, &errorMessage);
  if (!file) {
    llvm::errs() << errorMessage << "\n";
    return failure();
  }

```
- **EN**: Implements logic around `FileDescriptorIsDisplayed`, `errs`, `openInputFile`, `failure`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `FileDescriptorIsDisplayed`、`errs`、`openInputFile`、`failure` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

### Lines 830-845
```cpp
  auto output = openOutputFile(outputFilename, &errorMessage);
  if (!output) {
    llvm::errs() << errorMessage << "\n";
    return failure();
  }
  if (failed(MlirOptMain(output->os(), std::move(file), registry, config)))
    return failure();

  // Keep the output file if the invocation of MlirOptMain was successful.
  output->keep();
  return success();
}

LogicalResult mlir::MlirOptMain(int argc, char **argv, llvm::StringRef toolName,
                                DialectRegistry &registry) {

```
- **EN**: Implements logic around `openOutputFile`, `errs`, `failure`, `failed`, and 3 more symbols; this block makes success/failure or diagnostics explicit through MLIR result utilities.
- **CN**: 围绕 `openOutputFile`、`errs`、`failure`、`failed` 等另外 3 个符号 实现具体逻辑；该代码块通过 MLIR 结果工具显式表达成功/失败或诊断。

### Lines 846-852
```cpp
  // Register and parse command line options.
  std::string inputFilename, outputFilename;
  std::tie(inputFilename, outputFilename) =
      registerAndParseCLIOptions(argc, argv, toolName, registry);

  return MlirOptMain(argc, argv, inputFilename, outputFilename, registry);
}
```
- **EN**: Implements logic around `tie`, `registerAndParseCLIOptions`, `MlirOptMain`; this block parses or prints textual MLIR representations.
- **CN**: 围绕 `tie`、`registerAndParseCLIOptions`、`MlirOptMain` 实现具体逻辑；该代码块解析或打印文本形式的 MLIR 表示。

## Key Concepts / 关键概念

- **Tooling support / 工具链支持**:
  - **EN**: Builds reusable infrastructure for opt-like tools, translation drivers, and CLIs.
  - **CN**: 为类似 opt 的工具、翻译驱动和命令行程序构建可复用基础设施。
- **SSA and region-based IR / SSA 与基于 Region 的 IR**:
  - **EN**: The file works with MLIR operations, blocks, values, or regions that form MLIR's hierarchical SSA IR.
  - **CN**: 该文件处理 MLIR 操作、块、值或 Region，它们共同构成 MLIR 的层次化 SSA IR。
- **Structured diagnostics / 结构化诊断**:
  - **EN**: Failures are surfaced through `LogicalResult`, diagnostics, or related reporting helpers.
  - **CN**: 通过 `LogicalResult`、诊断对象或相关报告工具显式传播失败。
- **Pass pipelines / Pass 流水线**:
  - **EN**: The file participates in pass registration, scheduling, or execution orchestration.
  - **CN**: 该文件参与 Pass 的注册、调度或执行编排。
- **Bytecode support / Bytecode 支持**:
  - **EN**: The file reads, writes, or describes compact MLIR bytecode forms.
  - **CN**: 该文件读取、写出或描述紧凑的 MLIR bytecode 形式。
- **Streaming output / 流式输出**:
  - **EN**: Text, diagnostics, or generated content are emitted through LLVM stream APIs.
  - **CN**: 通过 LLVM 流 API 输出文本、诊断或生成内容。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir/Tools/mlir-opt/MlirOptMain.h`, `mlir/Bytecode/BytecodeWriter.h`, `mlir/Debug/CLOptionsSetup.h`, `mlir/Debug/Counter.h`, `mlir/Dialect/IRDL/IR/IRDL.h`, `mlir/Dialect/IRDL/IRDLLoading.h`, `mlir/IR/AsmState.h`, `mlir/IR/Attributes.h`, `mlir/IR/BuiltinOps.h`, `mlir/IR/Diagnostics.h` ... (+25 more)
- **Subsystem categories / 子系统类别**: LLVM support-library helpers / LLVM Support 库辅助工具 (10), core MLIR IR abstractions / 核心 MLIR IR 抽象 (7), tooling support declarations / 工具支持声明 (4), shared MLIR support helpers / 共享的 MLIR 支持工具 (3), pass-manager infrastructure / Pass 管理器基础设施 (2), MLIR bytecode serialization APIs / MLIR bytecode 序列化 API (1), MLIR parser declarations / MLIR 解析器声明 (1), LLVM ADT containers and utilities / LLVM ADT 容器与工具 (1)

# DependencyScanningTool.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Tooling/DependencyScanningTool.h`
- Repository: `llvm-project`
- Purpose (EN): The high-level implementation of the dependency discovery tool that runs on.
- 用途（中文）: 该文件为 Tooling 子系统中的 Dependency Scanning Tool 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21

```cpp
 1: //===----------------------------------------------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: 
 9: #ifndef LLVM_CLANG_TOOLING_DEPENDENCYSCANNINGTOOL_H
10: #define LLVM_CLANG_TOOLING_DEPENDENCYSCANNINGTOOL_H
11: 
12: #include "clang/DependencyScanning/DependencyScannerImpl.h"
13: #include "clang/DependencyScanning/DependencyScanningService.h"
14: #include "clang/DependencyScanning/DependencyScanningUtils.h"
15: #include "clang/DependencyScanning/DependencyScanningWorker.h"
16: #include "clang/DependencyScanning/ModuleDepCollector.h"
17: #include "clang/Tooling/CompilationDatabase.h"
18: #include "llvm/ADT/DenseSet.h"
19: #include <optional>
20: #include <string>
21: #include <vector>
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/DependencyScanning/DependencyScannerImpl.h`, `clang/DependencyScanning/DependencyScanningService.h`, `clang/DependencyScanning/DependencyScanningUtils.h` and 7 more.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/DependencyScanning/DependencyScannerImpl.h`, `clang/DependencyScanning/DependencyScanningService.h`, `clang/DependencyScanning/DependencyScanningUtils.h` 以及另外 7 项依赖。

### Lines 22-42

```cpp
22: 
23: namespace clang {
24: namespace tooling {
25: 
26: struct P1689Rule {
27:   std::string PrimaryOutput;
28:   std::optional<dependencies::P1689ModuleInfo> Provides;
29:   std::vector<dependencies::P1689ModuleInfo> Requires;
30: };
31: 
32: /// The high-level implementation of the dependency discovery tool that runs on
33: /// an individual worker thread.
34: class DependencyScanningTool {
35: public:
36:   /// Construct a dependency scanning tool.
37:   ///
38:   /// @param Service  The parent service. Must outlive the tool.
39:   DependencyScanningTool(dependencies::DependencyScanningService &Service)
40:       : Worker(Service) {}
41: 
42:   /// Print out the dependency information into a string using the dependency
```
- EN: It opens, closes, or documents namespace scope for `clang`, `tooling`. Key type declarations here include `P1689Rule`, `DependencyScanningTool`. It exposes API surface such as `Worker`.
- 中文: 它打开、关闭或说明了 `clang`, `tooling` 的命名空间作用域。 这里的重要类型声明包括 `P1689Rule`, `DependencyScanningTool`。 它暴露了 `Worker` 等接口。

### Lines 43-63

```cpp
43:   /// file format that is specified in the options (-MD is the default) and
44:   /// return it.
45:   ///
46:   /// \returns std::nullopt if errors occurred (reported to the DiagConsumer),
47:   /// dependency file contents otherwise.
48:   std::optional<std::string>
49:   getDependencyFile(ArrayRef<std::string> CommandLine, StringRef CWD,
50:                     dependencies::LookupModuleOutputCallback LookupModuleOutput,
51:                     DiagnosticConsumer &DiagConsumer);
52: 
53:   /// Collect the module dependency in P1689 format for C++20 named modules.
54:   ///
55:   /// \param MakeformatOutput The output parameter for dependency information
56:   /// in make format if the command line requires to generate make-format
57:   /// dependency information by `-MD -MF <dep_file>`.
58:   ///
59:   /// \param MakeformatOutputPath The output parameter for the path to
60:   /// \param MakeformatOutput.
61:   ///
62:   /// \returns std::nullopt if errors occurred (reported to the DiagConsumer),
63:   /// P1689 dependency format rules otherwise.
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 64-84

```cpp
64:   std::optional<P1689Rule>
65:   getP1689ModuleDependencyFile(const CompileCommand &Command, StringRef CWD,
66:                                std::string &MakeformatOutput,
67:                                std::string &MakeformatOutputPath,
68:                                DiagnosticConsumer &DiagConsumer);
69:   std::optional<P1689Rule>
70:   getP1689ModuleDependencyFile(const CompileCommand &Command, StringRef CWD,
71:                                DiagnosticConsumer &DiagConsumer) {
72:     std::string MakeformatOutput;
73:     std::string MakeformatOutputPath;
74: 
75:     return getP1689ModuleDependencyFile(Command, CWD, MakeformatOutput,
76:                                         MakeformatOutputPath, DiagConsumer);
77:   }
78: 
79:   /// Given a Clang driver command-line for a translation unit, gather the
80:   /// modular dependencies and return the information needed for explicit build.
81:   ///
82:   /// \param AlreadySeen This stores modules which have previously been
83:   ///                    reported. Use the same instance for all calls to this
84:   ///                    function for a single \c DependencyScanningTool in a
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 85-105

```cpp
 85:   ///                    single build. Use a different one for different tools,
 86:   ///                    and clear it between builds.
 87:   /// \param LookupModuleOutput This function is called to fill in
 88:   ///                           "-fmodule-file=", "-o" and other output
 89:   ///                           arguments for dependencies.
 90:   /// \param TUBuffer Optional memory buffer for translation unit input. If
 91:   ///                 TUBuffer is nullopt, the input should be included in the
 92:   ///                 Commandline already.
 93:   ///
 94:   /// \returns std::nullopt if errors occurred (reported to the DiagConsumer),
 95:   /// translation unit dependencies otherwise.
 96:   std::optional<dependencies::TranslationUnitDeps>
 97:   getTranslationUnitDependencies(
 98:       ArrayRef<std::string> CommandLine, StringRef CWD,
 99:       DiagnosticConsumer &DiagConsumer,
100:       const llvm::DenseSet<dependencies::ModuleID> &AlreadySeen,
101:       dependencies::LookupModuleOutputCallback LookupModuleOutput,
102:       std::optional<llvm::MemoryBufferRef> TUBuffer = std::nullopt);
103: 
104:   /// Given a compilation context specified via the Clang driver command-line,
105:   /// gather modular dependencies of module with the given name, and return the
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 106-126

```cpp
106:   /// information needed for explicit build.
107:   /// TODO: this method should be removed as soon as Swift and our C-APIs adopt
108:   /// CompilerInstanceWithContext. We are keeping it here so that it is easier
109:   /// to coordinate with Swift and C-API changes.
110:   llvm::Expected<dependencies::TranslationUnitDeps> getModuleDependencies(
111:       StringRef ModuleName, ArrayRef<std::string> CommandLine, StringRef CWD,
112:       const llvm::DenseSet<dependencies::ModuleID> &AlreadySeen,
113:       dependencies::DependencyActionController &Controller);
114: 
115:   /// Returns the worker tracing VFS, if it was requested via the service.
116:   llvm::vfs::TracingFileSystem *getWorkerTracingVFS() const {
117:     return Worker.getTracingVFS();
118:   }
119: 
120: private:
121:   dependencies::DependencyScanningWorker Worker;
122: 
123:   friend class CompilerInstanceWithContext;
124: };
125: 
126: /// Run the dependency scanning worker for the given driver or frontend
```
- EN: Key type declarations here include `CompilerInstanceWithContext`. It exposes API surface such as `getWorkerTracingVFS`, `getTracingVFS`.
- 中文: 这里的重要类型声明包括 `CompilerInstanceWithContext`。 它暴露了 `getWorkerTracingVFS`, `getTracingVFS` 等接口。

### Lines 127-147

```cpp
127: /// command-line, and report the discovered dependencies to the provided
128: /// consumer.
129: ///
130: /// OverlayFS should be based on the Worker's dependency scanning file-system
131: /// and can be used to provide any input specified on the command-line as
132: /// in-memory file. If no overlay file-system is provided, the Worker's
133: /// dependency scanning file-system is used instead.
134: ///
135: /// \returns false if any errors occurred (with diagnostics reported to
136: /// \c DiagConsumer), true otherwise.
137: bool computeDependencies(
138:     dependencies::DependencyScanningWorker &Worker, StringRef WorkingDirectory,
139:     ArrayRef<std::string> CommandLine,
140:     dependencies::DependencyConsumer &Consumer,
141:     dependencies::DependencyActionController &Controller,
142:     DiagnosticConsumer &DiagConsumer,
143:     llvm::IntrusiveRefCntPtr<llvm::vfs::OverlayFileSystem> OverlayFS = nullptr);
144: 
145: class CompilerInstanceWithContext {
146:   // Context
147:   dependencies::DependencyScanningWorker &Worker;
```
- EN: Key type declarations here include `CompilerInstanceWithContext`.
- 中文: 这里的重要类型声明包括 `CompilerInstanceWithContext`。

### Lines 148-168

```cpp
148:   llvm::StringRef CWD;
149:   std::vector<std::string> CommandLine;
150: 
151:   // Context - Diagnostics engine.
152:   DiagnosticConsumer *DiagConsumer = nullptr;
153:   std::unique_ptr<dependencies::DiagnosticsEngineWithDiagOpts>
154:       DiagEngineWithCmdAndOpts;
155:   std::unique_ptr<dependencies::TextDiagnosticsPrinterWithOutput>
156:       DiagPrinterWithOS;
157: 
158:   // Context - compiler invocation
159:   std::unique_ptr<CompilerInvocation> OriginalInvocation;
160: 
161:   // Context - output options
162:   std::unique_ptr<DependencyOutputOptions> OutputOpts;
163: 
164:   // Context - stable directory handling
165:   llvm::SmallVector<StringRef> StableDirs;
166:   dependencies::PrebuiltModulesAttrsMap PrebuiltModuleASTMap;
167: 
168:   // Compiler Instance
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 169-189

```cpp
169:   std::unique_ptr<CompilerInstance> CIPtr;
170: 
171:   // Source location offset.
172:   int32_t SrcLocOffset = 0;
173: 
174:   CompilerInstanceWithContext(dependencies::DependencyScanningWorker &Worker,
175:                               StringRef CWD,
176:                               const std::vector<std::string> &CMD)
177:       : Worker(Worker), CWD(CWD), CommandLine(CMD) {};
178: 
179:   bool initialize(dependencies::DependencyActionController &Controller,
180:                   std::unique_ptr<dependencies::DiagnosticsEngineWithDiagOpts>
181:                       DiagEngineWithDiagOpts,
182:                   IntrusiveRefCntPtr<llvm::vfs::OverlayFileSystem> OverlayFS);
183: 
184: public:
185:   /// @brief Initialize the tool's compiler instance from the commandline.
186:   ///        The compiler instance only takes a `-cc1` job, so this method
187:   ///        builds the `-cc1` job from the CommandLine input.
188:   /// @param Tool The dependency scanning tool whose compiler instance
189:   ///        with context is initialized.
```
- EN: It exposes API surface such as `Worker`.
- 中文: 它暴露了 `Worker` 等接口。

### Lines 190-210

```cpp
190:   /// @param CWD The current working directory.
191:   /// @param CommandLine This command line may be a driver command or a cc1
192:   ///        command.
193:   /// @param DC A diagnostics consumer to report error if the initialization
194:   ///        fails.
195:   static std::optional<CompilerInstanceWithContext> initializeFromCommandline(
196:       DependencyScanningTool &Tool, StringRef CWD,
197:       ArrayRef<std::string> CommandLine,
198:       dependencies::DependencyActionController &Controller,
199:       DiagnosticConsumer &DC);
200: 
201:   /// @brief Initializing the context and the compiler instance.
202:   ///        This method must be called before calling
203:   ///        computeDependenciesByNameWithContext.
204:   /// @param CWD The current working directory used during the scan.
205:   /// @param CommandLine The commandline used for the scan.
206:   /// @return Error if the initializaiton fails.
207:   static llvm::Expected<CompilerInstanceWithContext>
208:   initializeOrError(DependencyScanningTool &Tool, StringRef CWD,
209:                     ArrayRef<std::string> CommandLine,
210:                     dependencies::DependencyActionController &Controller);
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 211-231

```cpp
211: 
212:   bool
213:   computeDependencies(StringRef ModuleName,
214:                       dependencies::DependencyConsumer &Consumer,
215:                       dependencies::DependencyActionController &Controller);
216: 
217:   /// @brief Computes the dependeny for the module named ModuleName.
218:   /// @param ModuleName The name of the module for which this method computes
219:   ///.                  dependencies.
220:   /// @param AlreadySeen This stores modules which have previously been
221:   ///                    reported. Use the same instance for all calls to this
222:   ///                    function for a single \c DependencyScanningTool in a
223:   ///                    single build. Note that this parameter is not part of
224:   ///                    the context because it can be shared across different
225:   ///                    worker threads and each worker thread may update it.
226:   /// @param LookupModuleOutput This function is called to fill in
227:   ///                           "-fmodule-file=", "-o" and other output
228:   ///                           arguments for dependencies.
229:   /// @return An instance of \c TranslationUnitDeps if the scan is successful.
230:   ///         Otherwise it returns an error.
231:   llvm::Expected<dependencies::TranslationUnitDeps>
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 232-251

```cpp
232:   computeDependenciesByNameOrError(
233:       StringRef ModuleName,
234:       const llvm::DenseSet<dependencies::ModuleID> &AlreadySeen,
235:       dependencies::DependencyActionController &Controller);
236: 
237:   // MaxNumOfQueries is the upper limit of the number of names the by-name
238:   // scanning API (computeDependencies) can support after a
239:   // CompilerInstanceWithContext is initialized. At the time of this commit, the
240:   // estimated number of total unique importable names is around 3000 from
241:   // Apple's SDKs. We usually import them in parallel, so it is unlikely that
242:   // all names are all scanned by the same dependency scanning worker. Therefore
243:   // the 64k (20x bigger than our estimate) size is sufficient to hold the
244:   // unique source locations to report diagnostics per worker.
245:   static const int32_t MaxNumOfQueries = 1 << 16;
246: };
247: 
248: } // end namespace tooling
249: } // end namespace clang
250: 
251: #endif // LLVM_CLANG_TOOLING_DEPENDENCYSCANNINGTOOL_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `tooling`, `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `tooling`, `clang` 的命名空间作用域。

## Key Concepts / 关键概念

- `P1689Rule`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `DependencyScanningTool`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CompilerInstanceWithContext`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Worker`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getWorkerTracingVFS`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getTracingVFS`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/DependencyScanning/DependencyScannerImpl.h`, `clang/DependencyScanning/DependencyScanningService.h`, `clang/DependencyScanning/DependencyScanningUtils.h`, `clang/DependencyScanning/DependencyScanningWorker.h`, `clang/DependencyScanning/ModuleDepCollector.h`, `clang/Tooling/CompilationDatabase.h`, `llvm/ADT/DenseSet.h`, `optional`, `string`, `vector`
- Forward declarations / 前向声明: None / 无
- Namespace context / 命名空间上下文: `clang`, `tooling`
- Macro-style dependencies / 宏式依赖: None / 无

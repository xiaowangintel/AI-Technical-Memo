# Tooling.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Tooling/Tooling.h`
- Repository: `llvm-project`
- Purpose (EN): Framework for standalone Clang tools.
- 用途（中文）: 该文件为 Tooling 子系统中的 Tooling 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-45

```cpp
 1: //===- Tooling.h - Framework for standalone Clang tools ---------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //  This file implements functions to run clang tools standalone instead
10: //  of running them as a plugin.
11: //
12: //  A ClangTool is initialized with a CompilationDatabase and a set of files
13: //  to run over. The tool will then run a user-specified FrontendAction over
14: //  all TUs in which the given files are compiled.
15: //
16: //  It is also possible to run a FrontendAction over a snippet of code by
17: //  calling runToolOnCode, which is useful for unit testing.
18: //
19: //  Applications that need more fine grained control over how to run
20: //  multiple FrontendActions over code can use ToolInvocation.
21: //
22: //  Example tools:
23: //  - running clang -fsyntax-only over source code from an editor to get
24: //    fast syntax checks
25: //  - running match/replace tools over C++ code
26: //
27: //===----------------------------------------------------------------------===//
28: 
29: #ifndef LLVM_CLANG_TOOLING_TOOLING_H
30: #define LLVM_CLANG_TOOLING_TOOLING_H
31: 
32: #include "clang/AST/ASTConsumer.h"
33: #include "clang/Basic/FileManager.h"
34: #include "clang/Basic/LLVM.h"
35: #include "clang/Frontend/ASTUnit.h"
36: #include "clang/Frontend/FrontendAction.h"
37: #include "clang/Frontend/PCHContainerOperations.h"
38: #include "clang/Tooling/ArgumentsAdjusters.h"
39: #include "llvm/ADT/ArrayRef.h"
40: #include "llvm/ADT/IntrusiveRefCntPtr.h"
41: #include "llvm/ADT/StringMap.h"
42: #include "llvm/ADT/StringRef.h"
43: #include "llvm/ADT/StringSet.h"
44: #include "llvm/ADT/Twine.h"
45: #include "llvm/Option/Option.h"
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/AST/ASTConsumer.h`, `clang/Basic/FileManager.h`, `clang/Basic/LLVM.h` and 11 more.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/AST/ASTConsumer.h`, `clang/Basic/FileManager.h`, `clang/Basic/LLVM.h` 以及另外 11 项依赖。

### Lines 46-90

```cpp
46: #include "llvm/Support/VirtualFileSystem.h"
47: #include <memory>
48: #include <string>
49: #include <utility>
50: #include <vector>
51: 
52: namespace clang {
53: 
54: class CompilerInstance;
55: class CompilerInvocation;
56: class DiagnosticConsumer;
57: class DiagnosticsEngine;
58: 
59: namespace driver {
60: 
61: class Compilation;
62: 
63: } // namespace driver
64: 
65: namespace tooling {
66: 
67: class CompilationDatabase;
68: 
69: /// Retrieves the flags of the `-cc1` job in `Compilation` that has only source
70: /// files as its inputs.
71: /// Returns nullptr if there are no such jobs or multiple of them. Note that
72: /// offloading jobs are ignored.
73: const llvm::opt::ArgStringList *
74: getCC1Arguments(DiagnosticsEngine *Diagnostics,
75:                 driver::Compilation *Compilation);
76: 
77: /// Interface to process a clang::CompilerInvocation.
78: ///
79: /// If your tool is based on FrontendAction, you should be deriving from
80: /// FrontendActionFactory instead.
81: class ToolAction {
82: public:
83:   virtual ~ToolAction();
84: 
85:   /// Perform an action for an invocation.
86:   virtual bool
87:   runInvocation(std::shared_ptr<CompilerInvocation> Invocation,
88:                 FileManager *Files,
89:                 std::shared_ptr<PCHContainerOperations> PCHContainerOps,
90:                 DiagnosticConsumer *DiagConsumer) = 0;
```
- EN: This block imports dependencies such as `llvm/Support/VirtualFileSystem.h`, `memory`, `string` and 2 more. It opens, closes, or documents namespace scope for `clang`, `driver`, `tooling`. Key type declarations here include `CompilerInstance`, `CompilerInvocation`, `DiagnosticConsumer`, `DiagnosticsEngine`.
- 中文: 这一块引入了 `llvm/Support/VirtualFileSystem.h`, `memory`, `string` 以及另外 2 项依赖。 它打开、关闭或说明了 `clang`, `driver`, `tooling` 的命名空间作用域。 这里的重要类型声明包括 `CompilerInstance`, `CompilerInvocation`, `DiagnosticConsumer`, `DiagnosticsEngine`。

### Lines 91-135

```cpp
 91: };
 92: 
 93: /// Interface to generate clang::FrontendActions.
 94: ///
 95: /// Having a factory interface allows, for example, a new FrontendAction to be
 96: /// created for each translation unit processed by ClangTool.  This class is
 97: /// also a ToolAction which uses the FrontendActions created by create() to
 98: /// process each translation unit.
 99: class FrontendActionFactory : public ToolAction {
100: public:
101:   ~FrontendActionFactory() override;
102: 
103:   /// Invokes the compiler with a FrontendAction created by create().
104:   bool runInvocation(std::shared_ptr<CompilerInvocation> Invocation,
105:                      FileManager *Files,
106:                      std::shared_ptr<PCHContainerOperations> PCHContainerOps,
107:                      DiagnosticConsumer *DiagConsumer) override;
108: 
109:   /// Returns a new clang::FrontendAction.
110:   virtual std::unique_ptr<FrontendAction> create() = 0;
111: };
112: 
113: /// Returns a new FrontendActionFactory for a given type.
114: ///
115: /// T must derive from clang::FrontendAction.
116: ///
117: /// Example:
118: /// std::unique_ptr<FrontendActionFactory> Factory =
119: ///   newFrontendActionFactory<clang::SyntaxOnlyAction>();
120: template <typename T>
121: std::unique_ptr<FrontendActionFactory> newFrontendActionFactory();
122: 
123: /// Callbacks called before and after each source file processed by a
124: /// FrontendAction created by the FrontedActionFactory returned by \c
125: /// newFrontendActionFactory.
126: class SourceFileCallbacks {
127: public:
128:   virtual ~SourceFileCallbacks() = default;
129: 
130:   /// Called before a source file is processed by a FrontEndAction.
131:   /// \see clang::FrontendAction::BeginSourceFileAction
132:   virtual bool handleBeginSource(CompilerInstance &CI) {
133:     return true;
134:   }
135: 
```
- EN: Key type declarations here include `FrontendActionFactory`, `SourceFileCallbacks`. It exposes API surface such as `create`, `newFrontendActionFactory`, `~SourceFileCallbacks`, `handleBeginSource`.
- 中文: 这里的重要类型声明包括 `FrontendActionFactory`, `SourceFileCallbacks`。 它暴露了 `create`, `newFrontendActionFactory`, `~SourceFileCallbacks`, `handleBeginSource` 等接口。

### Lines 136-180

```cpp
136:   /// Called after a source file is processed by a FrontendAction.
137:   /// \see clang::FrontendAction::EndSourceFileAction
138:   virtual void handleEndSource() {}
139: };
140: 
141: /// Returns a new FrontendActionFactory for any type that provides an
142: /// implementation of newASTConsumer().
143: ///
144: /// FactoryT must implement: ASTConsumer *newASTConsumer().
145: ///
146: /// Example:
147: /// struct ProvidesASTConsumers {
148: ///   std::unique_ptr<clang::ASTConsumer> newASTConsumer();
149: /// } Factory;
150: /// std::unique_ptr<FrontendActionFactory> FactoryAdapter(
151: ///   newFrontendActionFactory(&Factory));
152: template <typename FactoryT>
153: inline std::unique_ptr<FrontendActionFactory> newFrontendActionFactory(
154:     FactoryT *ConsumerFactory, SourceFileCallbacks *Callbacks = nullptr);
155: 
156: /// Runs (and deletes) the tool on 'Code' with the -fsyntax-only flag.
157: ///
158: /// \param ToolAction The action to run over the code.
159: /// \param Code C++ code.
160: /// \param FileName The file name which 'Code' will be mapped as.
161: /// \param PCHContainerOps  The PCHContainerOperations for loading and creating
162: ///                         clang modules.
163: ///
164: /// \return - True if 'ToolAction' was successfully executed.
165: bool runToolOnCode(std::unique_ptr<FrontendAction> ToolAction, const Twine &Code,
166:                    const Twine &FileName = "input.cc",
167:                    std::shared_ptr<PCHContainerOperations> PCHContainerOps =
168:                        std::make_shared<PCHContainerOperations>());
169: 
170: /// The first part of the pair is the filename, the second part the
171: /// file-content.
172: using FileContentMappings = std::vector<std::pair<std::string, std::string>>;
173: 
174: /// Runs (and deletes) the tool on 'Code' with the -fsyntax-only flag and
175: ///        with additional other flags.
176: ///
177: /// \param ToolAction The action to run over the code.
178: /// \param Code C++ code.
179: /// \param Args Additional flags to pass on.
180: /// \param FileName The file name which 'Code' will be mapped as.
```
- EN: It defines convenient aliases such as `FileContentMappings`. It exposes API surface such as `handleEndSource`, `make_shared`.
- 中文: 它定义了 `FileContentMappings` 等便捷别名。 它暴露了 `handleEndSource`, `make_shared` 等接口。

### Lines 181-225

```cpp
181: /// \param ToolName The name of the binary running the tool. Standard library
182: ///                 header paths will be resolved relative to this.
183: /// \param PCHContainerOps   The PCHContainerOperations for loading and creating
184: ///                          clang modules.
185: ///
186: /// \return - True if 'ToolAction' was successfully executed.
187: bool runToolOnCodeWithArgs(
188:     std::unique_ptr<FrontendAction> ToolAction, const Twine &Code,
189:     const std::vector<std::string> &Args, const Twine &FileName = "input.cc",
190:     const Twine &ToolName = "clang-tool",
191:     std::shared_ptr<PCHContainerOperations> PCHContainerOps =
192:         std::make_shared<PCHContainerOperations>(),
193:     const FileContentMappings &VirtualMappedFiles = FileContentMappings());
194: 
195: // Similar to the overload except this takes a VFS.
196: bool runToolOnCodeWithArgs(
197:     std::unique_ptr<FrontendAction> ToolAction, const Twine &Code,
198:     llvm::IntrusiveRefCntPtr<llvm::vfs::FileSystem> VFS,
199:     const std::vector<std::string> &Args, const Twine &FileName = "input.cc",
200:     const Twine &ToolName = "clang-tool",
201:     std::shared_ptr<PCHContainerOperations> PCHContainerOps =
202:         std::make_shared<PCHContainerOperations>());
203: 
204: /// Builds an AST for 'Code'.
205: ///
206: /// \param Code C++ code.
207: /// \param FileName The file name which 'Code' will be mapped as.
208: /// \param PCHContainerOps The PCHContainerOperations for loading and creating
209: /// clang modules.
210: ///
211: /// \return The resulting AST or null if an error occurred.
212: std::unique_ptr<ASTUnit>
213: buildASTFromCode(StringRef Code, StringRef FileName = "input.cc",
214:                  std::shared_ptr<PCHContainerOperations> PCHContainerOps =
215:                      std::make_shared<PCHContainerOperations>());
216: 
217: /// Builds an AST for 'Code' with additional flags.
218: ///
219: /// \param Code C++ code.
220: /// \param Args Additional flags to pass on.
221: /// \param FileName The file name which 'Code' will be mapped as.
222: /// \param ToolName The name of the binary running the tool. Standard library
223: ///                 header paths will be resolved relative to this.
224: /// \param PCHContainerOps The PCHContainerOperations for loading and creating
225: /// clang modules.
```
- EN: It exposes API surface such as `FileContentMappings`, `make_shared`.
- 中文: 它暴露了 `FileContentMappings`, `make_shared` 等接口。

### Lines 226-270

```cpp
226: ///
227: /// \param Adjuster A function to filter the command line arguments as
228: /// specified.
229: ///
230: /// \param BaseFS FileSystem for managing and looking up files.
231: /// VirtualMappedFiles takes precedence.
232: ///
233: /// \return The resulting AST or null if an error occurred.
234: std::unique_ptr<ASTUnit> buildASTFromCodeWithArgs(
235:     StringRef Code, const std::vector<std::string> &Args,
236:     StringRef FileName = "input.cc", StringRef ToolName = "clang-tool",
237:     std::shared_ptr<PCHContainerOperations> PCHContainerOps =
238:         std::make_shared<PCHContainerOperations>(),
239:     ArgumentsAdjuster Adjuster = getClangStripDependencyFileAdjuster(),
240:     const FileContentMappings &VirtualMappedFiles = FileContentMappings(),
241:     DiagnosticConsumer *DiagConsumer = nullptr,
242:     IntrusiveRefCntPtr<llvm::vfs::FileSystem> BaseFS =
243:         llvm::vfs::getRealFileSystem(),
244:     CaptureDiagsKind CaptureKind = CaptureDiagsKind::None);
245: 
246: /// Utility to run a FrontendAction in a single clang invocation.
247: class ToolInvocation {
248: public:
249:   /// Create a tool invocation.
250:   ///
251:   /// \param CommandLine The command line arguments to clang. Note that clang
252:   /// uses its binary name (CommandLine[0]) to locate its builtin headers.
253:   /// Callers have to ensure that they are installed in a compatible location
254:   /// (see clang driver implementation) or mapped in via mapVirtualFile.
255:   /// \param FAction The action to be executed.
256:   /// \param Files The FileManager used for the execution. Class does not take
257:   /// ownership.
258:   /// \param PCHContainerOps The PCHContainerOperations for loading and creating
259:   /// clang modules.
260:   ToolInvocation(std::vector<std::string> CommandLine,
261:                  std::unique_ptr<FrontendAction> FAction, FileManager *Files,
262:                  std::shared_ptr<PCHContainerOperations> PCHContainerOps =
263:                      std::make_shared<PCHContainerOperations>());
264: 
265:   /// Create a tool invocation.
266:   ///
267:   /// \param CommandLine The command line arguments to clang.
268:   /// \param Action The action to be executed.
269:   /// \param Files The FileManager used for the execution.
270:   /// \param PCHContainerOps The PCHContainerOperations for loading and creating
```
- EN: Key type declarations here include `ToolInvocation`. It exposes API surface such as `make_shared`.
- 中文: 这里的重要类型声明包括 `ToolInvocation`。 它暴露了 `make_shared` 等接口。

### Lines 271-315

```cpp
271:   /// clang modules.
272:   ToolInvocation(std::vector<std::string> CommandLine, ToolAction *Action,
273:                  FileManager *Files,
274:                  std::shared_ptr<PCHContainerOperations> PCHContainerOps);
275: 
276:   ~ToolInvocation();
277: 
278:   ToolInvocation(const ToolInvocation &) = delete;
279:   ToolInvocation &operator=(const ToolInvocation &) = delete;
280: 
281:   /// Set a \c DiagnosticConsumer to use during driver command-line parsing and
282:   /// the action invocation itself.
283:   void setDiagnosticConsumer(DiagnosticConsumer *DiagConsumer) {
284:     this->DiagConsumer = DiagConsumer;
285:   }
286: 
287:   /// Set a \c DiagnosticOptions to use during driver command-line parsing.
288:   void setDiagnosticOptions(DiagnosticOptions *DiagOpts) {
289:     this->DiagOpts = DiagOpts;
290:   }
291: 
292:   /// Run the clang invocation.
293:   ///
294:   /// \returns True if there were no errors during execution.
295:   bool run();
296: 
297:  private:
298:   bool runInvocation(const char *BinaryName,
299:                      driver::Compilation *Compilation,
300:                      std::shared_ptr<CompilerInvocation> Invocation,
301:                      std::shared_ptr<PCHContainerOperations> PCHContainerOps);
302: 
303:   std::vector<std::string> CommandLine;
304:   ToolAction *Action;
305:   bool OwnsAction;
306:   FileManager *Files;
307:   std::shared_ptr<PCHContainerOperations> PCHContainerOps;
308:   DiagnosticConsumer *DiagConsumer = nullptr;
309:   DiagnosticOptions *DiagOpts = nullptr;
310: };
311: 
312: /// Utility to run a FrontendAction over a set of files.
313: ///
314: /// This class is written to be usable for command line utilities.
315: /// By default the class uses ClangSyntaxOnlyAdjuster to modify
```
- EN: It exposes API surface such as `~ToolInvocation`, `ToolInvocation`, `setDiagnosticConsumer`, `setDiagnosticOptions`.
- 中文: 它暴露了 `~ToolInvocation`, `ToolInvocation`, `setDiagnosticConsumer`, `setDiagnosticOptions` 等接口。

### Lines 316-360

```cpp
316: /// command line arguments before the arguments are used to run
317: /// a frontend action. One could install an additional command line
318: /// arguments adjuster by calling the appendArgumentsAdjuster() method.
319: class ClangTool {
320: public:
321:   /// Constructs a clang tool to run over a list of files.
322:   ///
323:   /// \param Compilations The CompilationDatabase which contains the compile
324:   ///        command lines for the given source paths.
325:   /// \param SourcePaths The source files to run over. If a source files is
326:   ///        not found in Compilations, it is skipped.
327:   /// \param PCHContainerOps The PCHContainerOperations for loading and creating
328:   /// clang modules.
329:   /// \param BaseFS VFS used for all underlying file accesses when running the
330:   /// tool.
331:   /// \param Files The file manager to use for underlying file operations when
332:   /// running the tool.
333:   ClangTool(const CompilationDatabase &Compilations,
334:             ArrayRef<std::string> SourcePaths,
335:             std::shared_ptr<PCHContainerOperations> PCHContainerOps =
336:                 std::make_shared<PCHContainerOperations>(),
337:             IntrusiveRefCntPtr<llvm::vfs::FileSystem> BaseFS =
338:                 llvm::vfs::getRealFileSystem(),
339:             IntrusiveRefCntPtr<FileManager> Files = nullptr);
340: 
341:   ~ClangTool();
342: 
343:   /// Set a \c DiagnosticConsumer to use during parsing.
344:   void setDiagnosticConsumer(DiagnosticConsumer *DiagConsumer) {
345:     this->DiagConsumer = DiagConsumer;
346:   }
347: 
348:   /// Map a virtual file to be used while running the tool.
349:   ///
350:   /// \param FilePath The path at which the content will be mapped.
351:   /// \param Content A null terminated buffer of the file's content.
352:   void mapVirtualFile(StringRef FilePath, StringRef Content);
353: 
354:   /// Append a command line arguments adjuster to the adjuster chain.
355:   ///
356:   /// \param Adjuster An argument adjuster, which will be run on the output of
357:   ///        previous argument adjusters.
358:   void appendArgumentsAdjuster(ArgumentsAdjuster Adjuster);
359: 
360:   /// Clear the command line arguments adjuster chain.
```
- EN: Key type declarations here include `ClangTool`. It exposes API surface such as `~ClangTool`, `setDiagnosticConsumer`, `mapVirtualFile`, `appendArgumentsAdjuster`.
- 中文: 这里的重要类型声明包括 `ClangTool`。 它暴露了 `~ClangTool`, `setDiagnosticConsumer`, `mapVirtualFile`, `appendArgumentsAdjuster` 等接口。

### Lines 361-405

```cpp
361:   void clearArgumentsAdjusters();
362: 
363:   /// Runs an action over all files specified in the command line.
364:   ///
365:   /// \param Action Tool action.
366:   ///
367:   /// \returns 0 on success; 1 if any error occurred; 2 if there is no error but
368:   /// some files are skipped due to missing compile commands.
369:   int run(ToolAction *Action);
370: 
371:   /// Create an AST for each file specified in the command line and
372:   /// append them to ASTs.
373:   int buildASTs(std::vector<std::unique_ptr<ASTUnit>> &ASTs);
374: 
375:   /// Sets whether an error message should be printed out if an action fails. By
376:   /// default, if an action fails, a message is printed out to stderr.
377:   void setPrintErrorMessage(bool PrintErrorMessage);
378: 
379:   /// Returns the file manager used in the tool.
380:   ///
381:   /// The file manager is shared between all translation units.
382:   FileManager &getFiles() { return *Files; }
383: 
384:   llvm::ArrayRef<std::string> getSourcePaths() const { return SourcePaths; }
385: 
386: private:
387:   const CompilationDatabase &Compilations;
388:   std::vector<std::string> SourcePaths;
389:   std::shared_ptr<PCHContainerOperations> PCHContainerOps;
390: 
391:   llvm::IntrusiveRefCntPtr<llvm::vfs::OverlayFileSystem> OverlayFileSystem;
392:   llvm::IntrusiveRefCntPtr<llvm::vfs::InMemoryFileSystem> InMemoryFileSystem;
393:   llvm::IntrusiveRefCntPtr<FileManager> Files;
394: 
395:   // Contains a list of pairs (<file name>, <file content>).
396:   std::vector<std::pair<StringRef, StringRef>> MappedFileContents;
397: 
398:   llvm::StringSet<> SeenWorkingDirectories;
399: 
400:   ArgumentsAdjuster ArgsAdjuster;
401: 
402:   DiagnosticConsumer *DiagConsumer = nullptr;
403: 
404:   bool PrintErrorMessage = true;
405: };
```
- EN: It exposes API surface such as `clearArgumentsAdjusters`, `run`, `buildASTs`, `setPrintErrorMessage`.
- 中文: 它暴露了 `clearArgumentsAdjusters`, `run`, `buildASTs`, `setPrintErrorMessage` 等接口。

### Lines 406-450

```cpp
406: 
407: template <typename T>
408: std::unique_ptr<FrontendActionFactory> newFrontendActionFactory() {
409:   class SimpleFrontendActionFactory : public FrontendActionFactory {
410:   public:
411:     std::unique_ptr<FrontendAction> create() override {
412:       return std::make_unique<T>();
413:     }
414:   };
415: 
416:   return std::unique_ptr<FrontendActionFactory>(
417:       new SimpleFrontendActionFactory);
418: }
419: 
420: template <typename FactoryT>
421: inline std::unique_ptr<FrontendActionFactory> newFrontendActionFactory(
422:     FactoryT *ConsumerFactory, SourceFileCallbacks *Callbacks) {
423:   class FrontendActionFactoryAdapter : public FrontendActionFactory {
424:   public:
425:     explicit FrontendActionFactoryAdapter(FactoryT *ConsumerFactory,
426:                                           SourceFileCallbacks *Callbacks)
427:         : ConsumerFactory(ConsumerFactory), Callbacks(Callbacks) {}
428: 
429:     std::unique_ptr<FrontendAction> create() override {
430:       return std::make_unique<ConsumerFactoryAdaptor>(ConsumerFactory,
431:                                                       Callbacks);
432:     }
433: 
434:   private:
435:     class ConsumerFactoryAdaptor : public ASTFrontendAction {
436:     public:
437:       ConsumerFactoryAdaptor(FactoryT *ConsumerFactory,
438:                              SourceFileCallbacks *Callbacks)
439:           : ConsumerFactory(ConsumerFactory), Callbacks(Callbacks) {}
440: 
441:       std::unique_ptr<ASTConsumer>
442:       CreateASTConsumer(CompilerInstance &, StringRef) override {
443:         return ConsumerFactory->newASTConsumer();
444:       }
445: 
446:     protected:
447:       bool BeginSourceFileAction(CompilerInstance &CI) override {
448:         if (!ASTFrontendAction::BeginSourceFileAction(CI))
449:           return false;
450:         if (Callbacks)
```
- EN: Key type declarations here include `SimpleFrontendActionFactory`, `FrontendActionFactoryAdapter`, `ConsumerFactoryAdaptor`. It exposes API surface such as `newFrontendActionFactory`, `make_unique`, `ConsumerFactory`, `newASTConsumer`.
- 中文: 这里的重要类型声明包括 `SimpleFrontendActionFactory`, `FrontendActionFactoryAdapter`, `ConsumerFactoryAdaptor`。 它暴露了 `newFrontendActionFactory`, `make_unique`, `ConsumerFactory`, `newASTConsumer` 等接口。

### Lines 451-495

```cpp
451:           return Callbacks->handleBeginSource(CI);
452:         return true;
453:       }
454: 
455:       void EndSourceFileAction() override {
456:         if (Callbacks)
457:           Callbacks->handleEndSource();
458:         ASTFrontendAction::EndSourceFileAction();
459:       }
460: 
461:     private:
462:       FactoryT *ConsumerFactory;
463:       SourceFileCallbacks *Callbacks;
464:     };
465:     FactoryT *ConsumerFactory;
466:     SourceFileCallbacks *Callbacks;
467:   };
468: 
469:   return std::unique_ptr<FrontendActionFactory>(
470:       new FrontendActionFactoryAdapter(ConsumerFactory, Callbacks));
471: }
472: 
473: /// Returns the absolute path of \c File, by prepending it with
474: /// the current directory if \c File is not absolute.
475: ///
476: /// Otherwise returns \c File.
477: /// If 'File' starts with "./", the returned path will not contain the "./".
478: /// Otherwise, the returned path will contain the literal path-concatenation of
479: /// the current directory and \c File.
480: ///
481: /// The difference to llvm::sys::fs::make_absolute is the canonicalization this
482: /// does by removing "./" and computing native paths.
483: ///
484: /// \param File Either an absolute or relative path.
485: std::string getAbsolutePath(StringRef File);
486: 
487: /// An overload of getAbsolutePath that works over the provided \p FS.
488: llvm::Expected<std::string> getAbsolutePath(llvm::vfs::FileSystem &FS,
489:                                             StringRef File);
490: 
491: /// Changes CommandLine to contain implicit flags that would have been
492: /// defined had the compiler driver been invoked through the path InvokedAs.
493: ///
494: /// For example, when called with \c InvokedAs set to `i686-linux-android-g++`,
495: /// the arguments '-target', 'i686-linux-android`, `--driver-mode=g++` will
```
- EN: It exposes API surface such as `handleBeginSource`, `handleEndSource`, `EndSourceFileAction`, `FrontendActionFactoryAdapter`.
- 中文: 它暴露了 `handleBeginSource`, `handleEndSource`, `EndSourceFileAction`, `FrontendActionFactoryAdapter` 等接口。

### Lines 496-529

```cpp
496: /// be inserted after the first argument in \c CommandLine.
497: ///
498: /// This function will not add new `-target` or `--driver-mode` flags if they
499: /// are already present in `CommandLine` (even if they have different settings
500: /// than would have been inserted).
501: ///
502: /// \pre `llvm::InitializeAllTargets()` has been called.
503: ///
504: /// \param CommandLine the command line used to invoke the compiler driver or
505: /// Clang tool, including the path to the executable as \c CommandLine[0].
506: /// \param InvokedAs the path to the driver used to infer implicit flags.
507: ///
508: /// \note This will not set \c CommandLine[0] to \c InvokedAs. The tooling
509: /// infrastructure expects that CommandLine[0] is a tool path relative to which
510: /// the builtin headers can be found.
511: void addTargetAndModeForProgramName(std::vector<std::string> &CommandLine,
512:                                     StringRef InvokedAs);
513: 
514: /// Helper function that expands response files in command line.
515: void addExpandedResponseFiles(std::vector<std::string> &CommandLine,
516:                               llvm::StringRef WorkingDir,
517:                               llvm::cl::TokenizerCallback Tokenizer,
518:                               llvm::vfs::FileSystem &FS);
519: 
520: /// Creates a \c CompilerInvocation.
521: CompilerInvocation *newInvocation(DiagnosticsEngine *Diagnostics,
522:                                   ArrayRef<const char *> CC1Args,
523:                                   const char *const BinaryName);
524: 
525: } // namespace tooling
526: 
527: } // namespace clang
528: 
529: #endif // LLVM_CLANG_TOOLING_TOOLING_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `tooling`, `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `tooling`, `clang` 的命名空间作用域。

## Key Concepts / 关键概念

- `CompilerInstance`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CompilerInvocation`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `DiagnosticConsumer`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `DiagnosticsEngine`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Compilation`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CompilationDatabase`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ToolAction`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `FrontendActionFactory`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/AST/ASTConsumer.h`, `clang/Basic/FileManager.h`, `clang/Basic/LLVM.h`, `clang/Frontend/ASTUnit.h`, `clang/Frontend/FrontendAction.h`, `clang/Frontend/PCHContainerOperations.h`, `clang/Tooling/ArgumentsAdjusters.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/IntrusiveRefCntPtr.h`, `llvm/ADT/StringMap.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSet.h`, `llvm/ADT/Twine.h`, `llvm/Option/Option.h`, `llvm/Support/VirtualFileSystem.h`
- Forward declarations / 前向声明: `CompilerInstance`, `CompilerInvocation`, `DiagnosticConsumer`, `DiagnosticsEngine`, `Compilation`, `CompilationDatabase`
- Namespace context / 命名空间上下文: `clang`, `driver`, `tooling`
- Macro-style dependencies / 宏式依赖: None / 无

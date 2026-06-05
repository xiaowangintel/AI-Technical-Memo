# CompilationDatabase.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Tooling/CompilationDatabase.h`
- Repository: `llvm-project`
- Purpose (EN): This file provides an interface and multiple implementations for CompilationDatabases. While C++ refactoring and analysis tools are not compilers, and thus don't run as part of the build system, they need the exact information.
- 用途（中文）: 该文件为 Tooling 子系统中的 Compilation Database 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21

```cpp
 1: //===- CompilationDatabase.h ------------------------------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //  This file provides an interface and multiple implementations for
10: //  CompilationDatabases.
11: //
12: //  While C++ refactoring and analysis tools are not compilers, and thus
13: //  don't run as part of the build system, they need the exact information
14: //  of a build in order to be able to correctly understand the C++ code of
15: //  the project. This information is provided via the CompilationDatabase
16: //  interface.
17: //
18: //  To create a CompilationDatabase from a build directory one can call
19: //  CompilationDatabase::loadFromDirectory(), which deduces the correct
20: //  compilation database from the root of the build tree.
21: //
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 22-42

```cpp
22: //  See the concrete subclasses of CompilationDatabase for currently supported
23: //  formats.
24: //
25: //===----------------------------------------------------------------------===//
26: 
27: #ifndef LLVM_CLANG_TOOLING_COMPILATIONDATABASE_H
28: #define LLVM_CLANG_TOOLING_COMPILATIONDATABASE_H
29: 
30: #include "clang/Basic/LLVM.h"
31: #include "llvm/ADT/ArrayRef.h"
32: #include "llvm/ADT/StringRef.h"
33: #include "llvm/ADT/Twine.h"
34: #include "llvm/Support/VirtualFileSystem.h"
35: #include <memory>
36: #include <string>
37: #include <utility>
38: #include <vector>
39: 
40: namespace clang {
41: namespace tooling {
42: 
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/Basic/LLVM.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringRef.h` and 6 more. It opens, closes, or documents namespace scope for `clang`, `tooling`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/Basic/LLVM.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringRef.h` 以及另外 6 项依赖。 它打开、关闭或说明了 `clang`, `tooling` 的命名空间作用域。

### Lines 43-63

```cpp
43: /// Specifies the working directory and command of a compilation.
44: struct CompileCommand {
45:   CompileCommand() = default;
46:   CompileCommand(const Twine &Directory, const Twine &Filename,
47:                  std::vector<std::string> CommandLine, const Twine &Output)
48:       : Directory(Directory.str()), Filename(Filename.str()),
49:         CommandLine(std::move(CommandLine)), Output(Output.str()) {}
50: 
51:   /// The working directory the command was executed from.
52:   std::string Directory;
53: 
54:   /// The source file associated with the command.
55:   std::string Filename;
56: 
57:   /// The command line that was executed.
58:   std::vector<std::string> CommandLine;
59: 
60:   /// The output file associated with the command.
61:   std::string Output;
62: 
63:   /// If this compile command was guessed rather than read from an authoritative
```
- EN: Key type declarations here include `CompileCommand`. It exposes API surface such as `CompileCommand`, `CommandLine`.
- 中文: 这里的重要类型声明包括 `CompileCommand`。 它暴露了 `CompileCommand`, `CommandLine` 等接口。

### Lines 64-84

```cpp
64:   /// source, a short human-readable explanation.
65:   /// e.g. "inferred from foo/bar.h".
66:   std::string Heuristic;
67: 
68:   friend bool operator==(const CompileCommand &LHS, const CompileCommand &RHS) {
69:     return LHS.Directory == RHS.Directory && LHS.Filename == RHS.Filename &&
70:            LHS.CommandLine == RHS.CommandLine && LHS.Output == RHS.Output &&
71:            LHS.Heuristic == RHS.Heuristic;
72:   }
73: 
74:   friend bool operator!=(const CompileCommand &LHS, const CompileCommand &RHS) {
75:     return !(LHS == RHS);
76:   }
77: };
78: 
79: /// Interface for compilation databases.
80: ///
81: /// A compilation database allows the user to retrieve compile command lines
82: /// for the files in a project.
83: ///
84: /// Many implementations are enumerable, allowing all command lines to be
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 85-105

```cpp
 85: /// retrieved. These can be used to run clang tools over a subset of the files
 86: /// in a project.
 87: class CompilationDatabase {
 88: public:
 89:   virtual ~CompilationDatabase();
 90: 
 91:   /// Loads a compilation database from a build directory.
 92:   ///
 93:   /// Looks at the specified 'BuildDirectory' and creates a compilation database
 94:   /// that allows to query compile commands for source files in the
 95:   /// corresponding source tree.
 96:   ///
 97:   /// Returns NULL and sets ErrorMessage if we were not able to build up a
 98:   /// compilation database for the build directory.
 99:   ///
100:   /// FIXME: Currently only supports JSON compilation databases, which
101:   /// are named 'compile_commands.json' in the given directory. Extend this
102:   /// for other build types (like ninja build files).
103:   static std::unique_ptr<CompilationDatabase>
104:   loadFromDirectory(StringRef BuildDirectory, std::string &ErrorMessage);
105: 
```
- EN: Key type declarations here include `CompilationDatabase`. It exposes API surface such as `~CompilationDatabase`, `loadFromDirectory`.
- 中文: 这里的重要类型声明包括 `CompilationDatabase`。 它暴露了 `~CompilationDatabase`, `loadFromDirectory` 等接口。

### Lines 106-126

```cpp
106:   /// Tries to detect a compilation database location and load it.
107:   ///
108:   /// Looks for a compilation database in all parent paths of file 'SourceFile'
109:   /// by calling loadFromDirectory.
110:   static std::unique_ptr<CompilationDatabase>
111:   autoDetectFromSource(StringRef SourceFile, std::string &ErrorMessage);
112: 
113:   /// Tries to detect a compilation database location and load it.
114:   ///
115:   /// Looks for a compilation database in directory 'SourceDir' and all
116:   /// its parent paths by calling loadFromDirectory.
117:   static std::unique_ptr<CompilationDatabase>
118:   autoDetectFromDirectory(StringRef SourceDir, std::string &ErrorMessage);
119: 
120:   /// Returns all compile commands in which the specified file was
121:   /// compiled.
122:   ///
123:   /// This includes compile commands that span multiple source files.
124:   /// For example, consider a project with the following compilations:
125:   /// $ clang++ -o test a.cc b.cc t.cc
126:   /// $ clang++ -o production a.cc b.cc -DPRODUCTION
```
- EN: It exposes API surface such as `autoDetectFromSource`, `autoDetectFromDirectory`.
- 中文: 它暴露了 `autoDetectFromSource`, `autoDetectFromDirectory` 等接口。

### Lines 127-147

```cpp
127:   /// A compilation database representing the project would return both command
128:   /// lines for a.cc and b.cc and only the first command line for t.cc.
129:   virtual std::vector<CompileCommand> getCompileCommands(
130:       StringRef FilePath) const = 0;
131: 
132:   /// Returns the list of all files available in the compilation database.
133:   ///
134:   /// By default, returns nothing. Implementations should override this if they
135:   /// can enumerate their source files.
136:   virtual std::vector<std::string> getAllFiles() const { return {}; }
137: 
138:   /// Returns all compile commands for all the files in the compilation
139:   /// database.
140:   ///
141:   /// FIXME: Add a layer in Tooling that provides an interface to run a tool
142:   /// over all files in a compilation database. Not all build systems have the
143:   /// ability to provide a feasible implementation for \c getAllCompileCommands.
144:   ///
145:   /// By default, this is implemented in terms of getAllFiles() and
146:   /// getCompileCommands(). Subclasses may override this for efficiency.
147:   virtual std::vector<CompileCommand> getAllCompileCommands() const;
```
- EN: It exposes API surface such as `getAllFiles`, `getAllCompileCommands`.
- 中文: 它暴露了 `getAllFiles`, `getAllCompileCommands` 等接口。

### Lines 148-168

```cpp
148: };
149: 
150: /// A compilation database that returns a single compile command line.
151: ///
152: /// Useful when we want a tool to behave more like a compiler invocation.
153: /// This compilation database is not enumerable: getAllFiles() returns {}.
154: class FixedCompilationDatabase : public CompilationDatabase {
155: public:
156:   /// Creates a FixedCompilationDatabase from the arguments after "--".
157:   ///
158:   /// Parses the given command line for "--". If "--" is found, the rest of
159:   /// the arguments will make up the command line in the returned
160:   /// FixedCompilationDatabase.
161:   /// The arguments after "--" must not include positional parameters or the
162:   /// argv[0] of the tool. Those will be added by the FixedCompilationDatabase
163:   /// when a CompileCommand is requested. The argv[0] of the returned command
164:   /// line will be "clang-tool".
165:   ///
166:   /// Returns NULL in case "--" is not found.
167:   ///
168:   /// The argument list is meant to be compatible with normal llvm command line
```
- EN: Key type declarations here include `FixedCompilationDatabase`.
- 中文: 这里的重要类型声明包括 `FixedCompilationDatabase`。

### Lines 169-189

```cpp
169:   /// parsing in main methods.
170:   /// int main(int argc, char **argv) {
171:   ///   std::unique_ptr<FixedCompilationDatabase> Compilations(
172:   ///     FixedCompilationDatabase::loadFromCommandLine(argc, argv));
173:   ///   cl::ParseCommandLineOptions(argc, argv);
174:   ///   ...
175:   /// }
176:   ///
177:   /// \param Argc The number of command line arguments - will be changed to
178:   /// the number of arguments before "--", if "--" was found in the argument
179:   /// list.
180:   /// \param Argv Points to the command line arguments.
181:   /// \param ErrorMsg Contains error text if the function returns null pointer.
182:   /// \param Directory The base directory used in the FixedCompilationDatabase.
183:   static std::unique_ptr<FixedCompilationDatabase>
184:   loadFromCommandLine(int &Argc, const char *const *Argv, std::string &ErrorMsg,
185:                       const Twine &Directory = ".");
186: 
187:   /// Reads flags from the given file, one-per-line.
188:   /// Returns nullptr and sets ErrorMessage if we can't read the file.
189:   static std::unique_ptr<FixedCompilationDatabase>
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 190-210

```cpp
190:   loadFromFile(StringRef Path, std::string &ErrorMsg);
191: 
192:   /// Reads flags from the given buffer, one-per-line.
193:   /// Directory is the command CWD, typically the parent of compile_flags.txt.
194:   static std::unique_ptr<FixedCompilationDatabase>
195:   loadFromBuffer(StringRef Directory, StringRef Data, std::string &ErrorMsg);
196: 
197:   /// Constructs a compilation data base from a specified directory
198:   /// and command line.
199:   FixedCompilationDatabase(const Twine &Directory,
200:                            ArrayRef<std::string> CommandLine);
201: 
202:   /// Returns the given compile command.
203:   ///
204:   /// Will always return a vector with one entry that contains the directory
205:   /// and command line specified at construction with "clang-tool" as argv[0]
206:   /// and 'FilePath' as positional argument.
207:   std::vector<CompileCommand>
208:   getCompileCommands(StringRef FilePath) const override;
209: 
210: private:
```
- EN: It exposes API surface such as `loadFromFile`, `loadFromBuffer`.
- 中文: 它暴露了 `loadFromFile`, `loadFromBuffer` 等接口。

### Lines 211-231

```cpp
211:   /// This is built up to contain a single entry vector to be returned from
212:   /// getCompileCommands after adding the positional argument.
213:   std::vector<CompileCommand> CompileCommands;
214: };
215: 
216: /// Transforms a compile command so that it applies the same configuration to
217: /// a different file. Most args are left intact, but tweaks may be needed
218: /// to certain flags (-x, -std etc).
219: ///
220: /// The output command will always end in {"--", Filename}.
221: tooling::CompileCommand transferCompileCommand(tooling::CompileCommand,
222:                                                StringRef Filename);
223: 
224: /// Returns a wrapped CompilationDatabase that defers to the provided one,
225: /// but getCompileCommands() will infer commands for unknown files.
226: /// The return value of getAllFiles() or getAllCompileCommands() is unchanged.
227: /// See InterpolatingCompilationDatabase.cpp for details on heuristics.
228: std::unique_ptr<CompilationDatabase>
229:     inferMissingCompileCommands(std::unique_ptr<CompilationDatabase>);
230: 
231: /// Returns a wrapped CompilationDatabase that will add -target and -mode flags
```
- EN: It exposes API surface such as `inferMissingCompileCommands`.
- 中文: 它暴露了 `inferMissingCompileCommands` 等接口。

### Lines 232-252

```cpp
232: /// to commandline when they can be deduced from argv[0] of commandline returned
233: /// by underlying database.
234: std::unique_ptr<CompilationDatabase>
235: inferTargetAndDriverMode(std::unique_ptr<CompilationDatabase> Base);
236: 
237: /// Returns a wrapped CompilationDatabase that will transform argv[0] to an
238: /// absolute path, if it currently is a plain tool name, looking it up in
239: /// PATH.
240: std::unique_ptr<CompilationDatabase>
241: inferToolLocation(std::unique_ptr<CompilationDatabase> Base);
242: 
243: /// Returns a wrapped CompilationDatabase that will expand all rsp(response)
244: /// files on commandline returned by underlying database.
245: std::unique_ptr<CompilationDatabase>
246: expandResponseFiles(std::unique_ptr<CompilationDatabase> Base,
247:                     llvm::IntrusiveRefCntPtr<llvm::vfs::FileSystem> FS);
248: 
249: } // namespace tooling
250: } // namespace clang
251: 
252: #endif // LLVM_CLANG_TOOLING_COMPILATIONDATABASE_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `tooling`, `clang`. It exposes API surface such as `inferTargetAndDriverMode`, `inferToolLocation`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `tooling`, `clang` 的命名空间作用域。 它暴露了 `inferTargetAndDriverMode`, `inferToolLocation` 等接口。

## Key Concepts / 关键概念

- `CompileCommand`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CompilationDatabase`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `FixedCompilationDatabase`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CommandLine`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `~CompilationDatabase`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `loadFromDirectory`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `autoDetectFromSource`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `autoDetectFromDirectory`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/Basic/LLVM.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/Support/VirtualFileSystem.h`, `memory`, `string`, `utility`, `vector`
- Forward declarations / 前向声明: None / 无
- Namespace context / 命名空间上下文: `clang`, `tooling`
- Macro-style dependencies / 宏式依赖: None / 无

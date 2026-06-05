# AtomicChange.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Tooling/Refactoring/AtomicChange.h`
- Repository: `llvm-project`
- Purpose (EN): AtomicChange class.
- 用途（中文）: 该文件为 Tooling::Refactoring 子系统中的 Atomic Change 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16

```cpp
 1: //===--- AtomicChange.h - AtomicChange class --------------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //  This file defines AtomicChange which is used to create a set of source
10: //  changes, e.g. replacements and header insertions.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #ifndef LLVM_CLANG_TOOLING_REFACTORING_ATOMICCHANGE_H
15: #define LLVM_CLANG_TOOLING_REFACTORING_ATOMICCHANGE_H
16: 
```
- EN: This range establishes include guards or other file-scope compilation boundaries.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。

### Lines 17-32

```cpp
17: #include "clang/Basic/SourceManager.h"
18: #include "clang/Format/Format.h"
19: #include "clang/Tooling/Core/Replacement.h"
20: #include "llvm/ADT/Any.h"
21: #include "llvm/ADT/StringRef.h"
22: #include "llvm/Support/Error.h"
23: 
24: namespace clang {
25: namespace tooling {
26: 
27: /// An atomic change is used to create and group a set of source edits,
28: /// e.g. replacements or header insertions. Edits in an AtomicChange should be
29: /// related, e.g. replacements for the same type reference and the corresponding
30: /// header insertion/deletion.
31: ///
32: /// An AtomicChange is uniquely identified by a key and will either be fully
```
- EN: This block imports dependencies such as `clang/Basic/SourceManager.h`, `clang/Format/Format.h`, `clang/Tooling/Core/Replacement.h` and 3 more. It opens, closes, or documents namespace scope for `clang`, `tooling`.
- 中文: 这一块引入了 `clang/Basic/SourceManager.h`, `clang/Format/Format.h`, `clang/Tooling/Core/Replacement.h` 以及另外 3 项依赖。 它打开、关闭或说明了 `clang`, `tooling` 的命名空间作用域。

### Lines 33-48

```cpp
33: /// applied or not applied at all.
34: ///
35: /// Calling setError on an AtomicChange stores the error message and marks it as
36: /// bad, i.e. none of its source edits will be applied.
37: class AtomicChange {
38: public:
39:   /// Creates an atomic change around \p KeyPosition with the key being a
40:   /// concatenation of the file name and the offset of \p KeyPosition.
41:   /// \p KeyPosition should be the location of the key syntactical element that
42:   /// is being changed, e.g. the call to a refactored method.
43:   AtomicChange(const SourceManager &SM, SourceLocation KeyPosition);
44: 
45:   AtomicChange(const SourceManager &SM, SourceLocation KeyPosition,
46:                llvm::Any Metadata);
47: 
48:   /// Creates an atomic change for \p FilePath with a customized key.
```
- EN: Key type declarations here include `AtomicChange`. It exposes API surface such as `AtomicChange`.
- 中文: 这里的重要类型声明包括 `AtomicChange`。 它暴露了 `AtomicChange` 等接口。

### Lines 49-64

```cpp
49:   AtomicChange(llvm::StringRef FilePath, llvm::StringRef Key)
50:       : Key(Key), FilePath(FilePath) {}
51: 
52:   AtomicChange(AtomicChange &&) = default;
53:   AtomicChange(const AtomicChange &) = default;
54: 
55:   AtomicChange &operator=(AtomicChange &&) = default;
56:   AtomicChange &operator=(const AtomicChange &) = default;
57: 
58:   bool operator==(const AtomicChange &Other) const;
59: 
60:   /// Returns the atomic change as a YAML string.
61:   std::string toYAMLString();
62: 
63:   /// Converts a YAML-encoded automic change to AtomicChange.
64:   static AtomicChange convertFromYAML(llvm::StringRef YAMLContent);
```
- EN: It exposes API surface such as `Key`, `AtomicChange`, `toYAMLString`, `convertFromYAML`.
- 中文: 它暴露了 `Key`, `AtomicChange`, `toYAMLString`, `convertFromYAML` 等接口。

### Lines 65-80

```cpp
65: 
66:   /// Returns the key of this change, which is a concatenation of the
67:   /// file name and offset of the key position.
68:   const std::string &getKey() const { return Key; }
69: 
70:   /// Returns the path of the file containing this atomic change.
71:   const std::string &getFilePath() const { return FilePath; }
72: 
73:   /// If this change could not be created successfully, e.g. because of
74:   /// conflicts among replacements, use this to set an error description.
75:   /// Thereby, places that cannot be fixed automatically can be gathered when
76:   /// applying changes.
77:   void setError(llvm::StringRef Error) { this->Error = std::string(Error); }
78: 
79:   /// Returns whether an error has been set on this list.
80:   bool hasError() const { return !Error.empty(); }
```
- EN: It exposes API surface such as `getKey`, `getFilePath`, `setError`, `hasError`.
- 中文: 它暴露了 `getKey`, `getFilePath`, `setError`, `hasError` 等接口。

### Lines 81-96

```cpp
81: 
82:   /// Returns the error message or an empty string if it does not exist.
83:   const std::string &getError() const { return Error; }
84: 
85:   /// Adds a replacement that replaces the given Range with
86:   /// ReplacementText.
87:   /// \returns An llvm::Error carrying ReplacementError on error.
88:   llvm::Error replace(const SourceManager &SM, const CharSourceRange &Range,
89:                       llvm::StringRef ReplacementText);
90: 
91:   /// Adds a replacement that replaces range [Loc, Loc+Length) with
92:   /// \p Text.
93:   /// \returns An llvm::Error carrying ReplacementError on error.
94:   llvm::Error replace(const SourceManager &SM, SourceLocation Loc,
95:                       unsigned Length, llvm::StringRef Text);
96: 
```
- EN: It exposes API surface such as `getError`.
- 中文: 它暴露了 `getError` 等接口。

### Lines 97-112

```cpp
 97:   /// Adds a replacement that inserts \p Text at \p Loc. If this
 98:   /// insertion conflicts with an existing insertion (at the same position),
 99:   /// this will be inserted before/after the existing insertion depending on
100:   /// \p InsertAfter. Users should use `replace` with `Length=0` instead if they
101:   /// do not want conflict resolving by default. If the conflicting replacement
102:   /// is not an insertion, an error is returned.
103:   ///
104:   /// \returns An llvm::Error carrying ReplacementError on error.
105:   llvm::Error insert(const SourceManager &SM, SourceLocation Loc,
106:                      llvm::StringRef Text, bool InsertAfter = true);
107: 
108:   /// Adds a header into the file that contains the key position.
109:   /// Header can be in angle brackets or double quotation marks. By default
110:   /// (header is not quoted), header will be surrounded with double quotes.
111:   void addHeader(llvm::StringRef Header);
112: 
```
- EN: It exposes API surface such as `addHeader`.
- 中文: 它暴露了 `addHeader` 等接口。

### Lines 113-128

```cpp
113:   /// Removes a header from the file that contains the key position.
114:   void removeHeader(llvm::StringRef Header);
115: 
116:   /// Returns a const reference to existing replacements.
117:   const Replacements &getReplacements() const { return Replaces; }
118: 
119:   Replacements &getReplacements() { return Replaces; }
120: 
121:   llvm::ArrayRef<std::string> getInsertedHeaders() const {
122:     return InsertedHeaders;
123:   }
124: 
125:   llvm::ArrayRef<std::string> getRemovedHeaders() const {
126:     return RemovedHeaders;
127:   }
128: 
```
- EN: It exposes API surface such as `removeHeader`, `getReplacements`, `getInsertedHeaders`, `getRemovedHeaders`.
- 中文: 它暴露了 `removeHeader`, `getReplacements`, `getInsertedHeaders`, `getRemovedHeaders` 等接口。

### Lines 129-144

```cpp
129:   const llvm::Any &getMetadata() const { return Metadata; }
130: 
131: private:
132:   AtomicChange() {}
133: 
134:   AtomicChange(std::string Key, std::string FilePath, std::string Error,
135:                std::vector<std::string> InsertedHeaders,
136:                std::vector<std::string> RemovedHeaders,
137:                clang::tooling::Replacements Replaces);
138: 
139:   // This uniquely identifies an AtomicChange.
140:   std::string Key;
141:   std::string FilePath;
142:   std::string Error;
143:   std::vector<std::string> InsertedHeaders;
144:   std::vector<std::string> RemovedHeaders;
```
- EN: It exposes API surface such as `getMetadata`, `AtomicChange`.
- 中文: 它暴露了 `getMetadata`, `AtomicChange` 等接口。

### Lines 145-160

```cpp
145:   tooling::Replacements Replaces;
146: 
147:   // This field stores metadata which is ignored for the purposes of applying
148:   // edits to source, but may be useful for other consumers of AtomicChanges. In
149:   // particular, consumers can use this to direct how they want to consume each
150:   // edit.
151:   llvm::Any Metadata;
152: };
153: 
154: using AtomicChanges = std::vector<AtomicChange>;
155: 
156: // Defines specs for applying changes.
157: struct ApplyChangesSpec {
158:   // If true, cleans up redundant/erroneous code around changed code with
159:   // clang-format's cleanup functionality, e.g. redundant commas around deleted
160:   // parameter or empty namespaces introduced by deletions.
```
- EN: Key type declarations here include `ApplyChangesSpec`. It defines convenient aliases such as `AtomicChanges`.
- 中文: 这里的重要类型声明包括 `ApplyChangesSpec`。 它定义了 `AtomicChanges` 等便捷别名。

### Lines 161-176

```cpp
161:   bool Cleanup = true;
162: 
163:   format::FormatStyle Style = format::getNoStyle();
164: 
165:   // Options for selectively formatting changes with clang-format:
166:   // kAll: Format all changed lines.
167:   // kNone: Don't format anything.
168:   // kViolations: Format lines exceeding the `ColumnLimit` in `Style`.
169:   enum FormatOption { kAll, kNone, kViolations };
170: 
171:   FormatOption Format = kNone;
172: };
173: 
174: /// Applies all AtomicChanges in \p Changes to the \p Code.
175: ///
176: /// This completely ignores the file path in each change and replaces them with
```
- EN: It introduces enum-based state or option sets such as `FormatOption`. It exposes API surface such as `getNoStyle`.
- 中文: 它引入了 `FormatOption` 等基于枚举的状态或选项集合。 它暴露了 `getNoStyle` 等接口。

### Lines 177-192

```cpp
177: /// \p FilePath, i.e. callers are responsible for ensuring all changes are for
178: /// the same file.
179: ///
180: /// \returns The changed code if all changes are applied successfully;
181: /// otherwise, an llvm::Error carrying llvm::StringError is returned (the Error
182: /// message can be converted to string with `llvm::toString()` and the
183: /// error_code should be ignored).
184: llvm::Expected<std::string>
185: applyAtomicChanges(llvm::StringRef FilePath, llvm::StringRef Code,
186:                    llvm::ArrayRef<AtomicChange> Changes,
187:                    const ApplyChangesSpec &Spec);
188: 
189: } // end namespace tooling
190: } // end namespace clang
191: 
192: #endif // LLVM_CLANG_TOOLING_REFACTORING_ATOMICCHANGE_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `tooling`, `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `tooling`, `clang` 的命名空间作用域。

## Key Concepts / 关键概念

- `AtomicChange`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `AtomicChanges`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。
- `ApplyChangesSpec`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `FormatOption`: An enumeration that captures a bounded set of modes, states, or categories. / 用于表达有限模式、状态或类别的枚举。
- `Key`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `toYAMLString`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `convertFromYAML`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getKey`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/Basic/SourceManager.h`, `clang/Format/Format.h`, `clang/Tooling/Core/Replacement.h`, `llvm/ADT/Any.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Error.h`
- Forward declarations / 前向声明: None / 无
- Namespace context / 命名空间上下文: `clang`, `tooling`
- Macro-style dependencies / 宏式依赖: None / 无

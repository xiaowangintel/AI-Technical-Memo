# CallDescription.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/StaticAnalyzer/Core/PathSensitive/CallDescription.h`
- Repository: `llvm-project`
- Purpose (EN): function/method call matching.
- 用途（中文）: 该文件为 StaticAnalyzer::Core::PathSensitive 子系统中的 Call Description 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25

```cpp
 1: //===- CallDescription.h - function/method call matching       --*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: /// \file This file defines a generic mechanism for matching for function and
10: /// method calls of C, C++, and Objective-C languages. Instances of these
11: /// classes are frequently used together with the CallEvent classes.
12: //
13: //===----------------------------------------------------------------------===//
14: 
15: #ifndef LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_CALLDESCRIPTION_H
16: #define LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_CALLDESCRIPTION_H
17: 
18: #include "clang/StaticAnalyzer/Core/PathSensitive/CallEvent.h"
19: #include "llvm/ADT/ArrayRef.h"
20: #include "llvm/Support/Compiler.h"
21: #include <optional>
22: #include <vector>
23: 
24: namespace clang {
25: class IdentifierInfo;
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/StaticAnalyzer/Core/PathSensitive/CallEvent.h`, `llvm/ADT/ArrayRef.h`, `llvm/Support/Compiler.h` and 2 more. It opens, closes, or documents namespace scope for `clang`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/StaticAnalyzer/Core/PathSensitive/CallEvent.h`, `llvm/ADT/ArrayRef.h`, `llvm/Support/Compiler.h` 以及另外 2 项依赖。 它打开、关闭或说明了 `clang` 的命名空间作用域。

### Lines 26-50

```cpp
26: } // namespace clang
27: 
28: namespace clang {
29: namespace ento {
30: /// A `CallDescription` is a pattern that can be used to _match_ calls
31: /// based on the qualified name and the argument/parameter counts.
32: class CallDescription {
33: public:
34:   enum class Mode {
35:     /// Match calls to functions from the C standard library. This also
36:     /// recognizes builtin variants whose name is derived by adding
37:     /// "__builtin", "__inline" or similar prefixes or suffixes; but only
38:     /// matches functions that are externally visible and are declared either
39:     /// directly within a TU or in the namespace 'std'.
40:     /// For the exact heuristics, see CheckerContext::isCLibraryFunction().
41:     CLibrary,
42: 
43:     /// An extended version of the `CLibrary` mode that also matches the
44:     /// hardened variants like __FOO_chk() and __builtin__FOO_chk() that take
45:     /// additional arguments compared to the "regular" function FOO().
46:     /// This is not the default behavior of `CLibrary` because in this case the
47:     /// checker code must be prepared to handle the different parametrization.
48:     /// For the exact heuristics, see CheckerContext::isHardenedVariantOf().
49:     CLibraryMaybeHardened,
50: 
```
- EN: It opens, closes, or documents namespace scope for `clang`, `ento`. Key type declarations here include `CallDescription`, `Mode`. It introduces enum-based state or option sets such as `Mode`.
- 中文: 它打开、关闭或说明了 `clang`, `ento` 的命名空间作用域。 这里的重要类型声明包括 `CallDescription`, `Mode`。 它引入了 `Mode` 等基于枚举的状态或选项集合。

### Lines 51-75

```cpp
51:     /// Matches "simple" functions that are not methods. (Static methods are
52:     /// methods.)
53:     SimpleFunc,
54: 
55:     /// Matches a C++ method (may be static, may be virtual, may be an
56:     /// overloaded operator, a constructor or a destructor).
57:     CXXMethod,
58: 
59:     /// Match any CallEvent that is not an ObjCMethodCall. This should not be
60:     /// used when the checker looks for a concrete function (and knows whether
61:     /// it is a method); but GenericTaintChecker uses this mode to match
62:     /// functions whose name was configured by the user.
63:     Unspecified,
64: 
65:     /// FIXME: Add support for ObjCMethodCall events (I'm not adding it because
66:     /// I'm not familiar with Objective-C). Note that currently an early return
67:     /// in `bool matches(const CallEvent &Call) const;` discards all
68:     /// Objective-C method calls.
69:   };
70: 
71: private:
72:   friend class CallEvent;
73:   using MaybeCount = std::optional<unsigned>;
74: 
75:   mutable std::optional<const IdentifierInfo *> II;
```
- EN: Key type declarations here include `CallEvent`. It defines convenient aliases such as `MaybeCount`.
- 中文: 这里的重要类型声明包括 `CallEvent`。 它定义了 `MaybeCount` 等便捷别名。

### Lines 76-100

```cpp
 76:   // The list of the qualified names used to identify the specified CallEvent,
 77:   // e.g. "{a, b}" represent the qualified names, like "a::b".
 78:   std::vector<std::string> QualifiedName;
 79:   MaybeCount RequiredArgs;
 80:   MaybeCount RequiredParams;
 81:   Mode MatchAs;
 82: 
 83: public:
 84:   /// Constructs a CallDescription object.
 85:   ///
 86:   /// @param MatchAs Specifies the kind of the call that should be matched.
 87:   ///
 88:   /// @param QualifiedName The list of the name qualifiers of the function that
 89:   /// will be matched. The user is allowed to skip any of the qualifiers.
 90:   /// For example, {"std", "basic_string", "c_str"} would match both
 91:   /// std::basic_string<...>::c_str() and std::__1::basic_string<...>::c_str().
 92:   ///
 93:   /// @param RequiredArgs The expected number of arguments that are passed to
 94:   /// the function. Omit this parameter (or pass std::nullopt) to match every
 95:   /// occurrence without checking the argument count in the call.
 96:   ///
 97:   /// @param RequiredParams The expected number of parameters in the function
 98:   /// definition that is called. Omit this parameter to match every occurrence
 99:   /// without checking the parameter count in the definition.
100:   CallDescription(Mode MatchAs, ArrayRef<StringRef> QualifiedName,
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 101-125

```cpp
101:                   MaybeCount RequiredArgs = std::nullopt,
102:                   MaybeCount RequiredParams = std::nullopt);
103: 
104:   /// Get the name of the function that this object matches.
105:   StringRef getFunctionName() const { return QualifiedName.back(); }
106: 
107:   /// Get the qualified name parts in reversed order.
108:   /// E.g. { "std", "vector", "data" } -> "vector", "std"
109:   auto begin_qualified_name_parts() const {
110:     return std::next(QualifiedName.rbegin());
111:   }
112:   auto end_qualified_name_parts() const { return QualifiedName.rend(); }
113: 
114:   /// It's false, if and only if we expect a single identifier, such as
115:   /// `getenv`. It's true for `std::swap`, or `my::detail::container::data`.
116:   bool hasQualifiedNameParts() const { return QualifiedName.size() > 1; }
117: 
118:   /// @name Matching CallDescriptions against a CallEvent
119:   /// @{
120: 
121:   /// Returns true if the CallEvent is a call to a function that matches
122:   /// the CallDescription.
123:   ///
124:   /// \note This function is not intended to be used to match Obj-C method
125:   /// calls.
```
- EN: It exposes API surface such as `getFunctionName`, `begin_qualified_name_parts`, `next`, `end_qualified_name_parts`.
- 中文: 它暴露了 `getFunctionName`, `begin_qualified_name_parts`, `next`, `end_qualified_name_parts` 等接口。

### Lines 126-150

```cpp
126:   bool matches(const CallEvent &Call) const;
127: 
128:   /// Returns true whether the CallEvent matches on any of the CallDescriptions
129:   /// supplied.
130:   ///
131:   /// \note This function is not intended to be used to match Obj-C method
132:   /// calls.
133:   friend bool matchesAny(const CallEvent &Call, const CallDescription &CD1) {
134:     return CD1.matches(Call);
135:   }
136: 
137:   /// \copydoc clang::ento::CallDescription::matchesAny(const CallEvent &, const CallDescription &)
138:   template <typename... Ts>
139:   friend bool matchesAny(const CallEvent &Call, const CallDescription &CD1,
140:                          const Ts &...CDs) {
141:     return CD1.matches(Call) || matchesAny(Call, CDs...);
142:   }
143:   /// @}
144: 
145:   /// @name Matching CallDescriptions against a CallExpr
146:   /// @{
147: 
148:   /// Returns true if the CallExpr is a call to a function that matches the
149:   /// CallDescription.
150:   ///
```
- EN: It exposes API surface such as `matches`, `matchesAny`.
- 中文: 它暴露了 `matches`, `matchesAny` 等接口。

### Lines 151-175

```cpp
151:   /// When available, always prefer matching with a CallEvent! This function
152:   /// exists only when that is not available, for example, when _only_
153:   /// syntactic check is done on a piece of code.
154:   ///
155:   /// Also, StdLibraryFunctionsChecker::Signature is likely a better candidate
156:   /// for syntactic only matching if you are writing a new checker. This is
157:   /// handy if a CallDescriptionMap is already there.
158:   ///
159:   /// The function is imprecise because CallEvent may know path sensitive
160:   /// information, such as the precise argument count (see comments for
161:   /// CallEvent::getNumArgs), the called function if it was called through a
162:   /// function pointer, and other information not available syntactically.
163:   bool matchesAsWritten(const CallExpr &CE) const;
164: 
165:   /// Returns true whether the CallExpr matches on any of the CallDescriptions
166:   /// supplied.
167:   ///
168:   /// \note This function is not intended to be used to match Obj-C method
169:   /// calls.
170:   friend bool matchesAnyAsWritten(const CallExpr &CE,
171:                                   const CallDescription &CD1) {
172:     return CD1.matchesAsWritten(CE);
173:   }
174: 
175:   /// \copydoc clang::ento::CallDescription::matchesAnyAsWritten(const CallExpr &, const CallDescription &)
```
- EN: It exposes API surface such as `matchesAsWritten`.
- 中文: 它暴露了 `matchesAsWritten` 等接口。

### Lines 176-200

```cpp
176:   template <typename... Ts>
177:   friend bool matchesAnyAsWritten(const CallExpr &CE,
178:                                   const CallDescription &CD1,
179:                                   const Ts &...CDs) {
180:     return CD1.matchesAsWritten(CE) || matchesAnyAsWritten(CE, CDs...);
181:   }
182:   /// @}
183: 
184: private:
185:   bool matchesImpl(const FunctionDecl *Callee, size_t ArgCount,
186:                    size_t ParamCount) const;
187: 
188:   bool matchNameOnly(const NamedDecl *ND) const;
189:   bool matchQualifiedNameParts(const Decl *D) const;
190: };
191: 
192: /// An immutable map from CallDescriptions to arbitrary data. Provides a unified
193: /// way for checkers to react on function calls.
194: template <typename T> class CallDescriptionMap {
195:   friend class CallDescriptionSet;
196: 
197:   // Some call descriptions aren't easily hashable (eg., the ones with qualified
198:   // names in which some sections are omitted), so let's put them
199:   // in a simple vector and use linear lookup.
200:   // TODO: Implement an actual map for fast lookup for "hashable" call
```
- EN: Key type declarations here include `CallDescriptionMap`, `CallDescriptionSet`. It exposes API surface such as `matchesAsWritten`, `matchNameOnly`, `matchQualifiedNameParts`.
- 中文: 这里的重要类型声明包括 `CallDescriptionMap`, `CallDescriptionSet`。 它暴露了 `matchesAsWritten`, `matchNameOnly`, `matchQualifiedNameParts` 等接口。

### Lines 201-225

```cpp
201:   // descriptions (eg., the ones for C functions that just match the name).
202:   std::vector<std::pair<CallDescription, T>> LinearMap;
203: 
204: public:
205:   CallDescriptionMap(
206:       std::initializer_list<std::pair<CallDescription, T>> &&List)
207:       : LinearMap(List) {}
208: 
209:   template <typename InputIt>
210:   CallDescriptionMap(InputIt First, InputIt Last) : LinearMap(First, Last) {}
211: 
212:   ~CallDescriptionMap() = default;
213: 
214:   // These maps are usually stored once per checker, so let's make sure
215:   // we don't do redundant copies.
216:   CallDescriptionMap(const CallDescriptionMap &) = delete;
217:   CallDescriptionMap &operator=(const CallDescription &) = delete;
218: 
219:   CallDescriptionMap(CallDescriptionMap &&) = default;
220:   CallDescriptionMap &operator=(CallDescriptionMap &&) = default;
221: 
222:   [[nodiscard]] const T *lookup(const CallEvent &Call) const {
223:     // Slow path: linear lookup.
224:     // TODO: Implement some sort of fast path.
225:     for (const std::pair<CallDescription, T> &I : LinearMap)
```
- EN: It exposes API surface such as `LinearMap`, `CallDescriptionMap`, `~CallDescriptionMap`, `lookup`.
- 中文: 它暴露了 `LinearMap`, `CallDescriptionMap`, `~CallDescriptionMap`, `lookup` 等接口。

### Lines 226-250

```cpp
226:       if (I.first.matches(Call))
227:         return &I.second;
228: 
229:     return nullptr;
230:   }
231: 
232:   /// When available, always prefer lookup with a CallEvent! This function
233:   /// exists only when that is not available, for example, when _only_
234:   /// syntactic check is done on a piece of code.
235:   ///
236:   /// Also, StdLibraryFunctionsChecker::Signature is likely a better candidate
237:   /// for syntactic only matching if you are writing a new checker. This is
238:   /// handy if a CallDescriptionMap is already there.
239:   ///
240:   /// The function is imprecise because CallEvent may know path sensitive
241:   /// information, such as the precise argument count (see comments for
242:   /// CallEvent::getNumArgs), the called function if it was called through a
243:   /// function pointer, and other information not available syntactically.
244:   [[nodiscard]] const T *lookupAsWritten(const CallExpr &Call) const {
245:     // Slow path: linear lookup.
246:     // TODO: Implement some sort of fast path.
247:     for (const std::pair<CallDescription, T> &I : LinearMap)
248:       if (I.first.matchesAsWritten(Call))
249:         return &I.second;
250: 
```
- EN: It exposes API surface such as `lookupAsWritten`.
- 中文: 它暴露了 `lookupAsWritten` 等接口。

### Lines 251-275

```cpp
251:     return nullptr;
252:   }
253: };
254: 
255: /// Enumerators of this enum class are used to construct CallDescription
256: /// objects; in that context the fully qualified name is needlessly verbose.
257: using CDM = CallDescription::Mode;
258: 
259: /// An immutable set of CallDescriptions.
260: /// Checkers can efficiently decide if a given CallEvent matches any
261: /// CallDescription in the set.
262: class CallDescriptionSet {
263:   CallDescriptionMap<bool /*unused*/> Impl = {};
264: 
265: public:
266:   CallDescriptionSet(std::initializer_list<CallDescription> &&List);
267: 
268:   CallDescriptionSet(const CallDescriptionSet &) = delete;
269:   CallDescriptionSet &operator=(const CallDescription &) = delete;
270: 
271:   [[nodiscard]] bool contains(const CallEvent &Call) const;
272: 
273:   /// When available, always prefer lookup with a CallEvent! This function
274:   /// exists only when that is not available, for example, when _only_
275:   /// syntactic check is done on a piece of code.
```
- EN: Key type declarations here include `CallDescriptionSet`. It defines convenient aliases such as `CDM`. It exposes API surface such as `CallDescriptionSet`, `contains`.
- 中文: 这里的重要类型声明包括 `CallDescriptionSet`。 它定义了 `CDM` 等便捷别名。 它暴露了 `CallDescriptionSet`, `contains` 等接口。

### Lines 276-291

```cpp
276:   ///
277:   /// Also, StdLibraryFunctionsChecker::Signature is likely a better candidate
278:   /// for syntactic only matching if you are writing a new checker. This is
279:   /// handy if a CallDescriptionMap is already there.
280:   ///
281:   /// The function is imprecise because CallEvent may know path sensitive
282:   /// information, such as the precise argument count (see comments for
283:   /// CallEvent::getNumArgs), the called function if it was called through a
284:   /// function pointer, and other information not available syntactically.
285:   [[nodiscard]] bool containsAsWritten(const CallExpr &CE) const;
286: };
287: 
288: } // namespace ento
289: } // namespace clang
290: 
291: #endif // LLVM_CLANG_STATICANALYZER_CORE_PATHSENSITIVE_CALLDESCRIPTION_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `ento`, `clang`. It exposes API surface such as `containsAsWritten`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `ento`, `clang` 的命名空间作用域。 它暴露了 `containsAsWritten` 等接口。

## Key Concepts / 关键概念

- `IdentifierInfo`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CallDescription`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Mode`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CallEvent`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `MaybeCount`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。
- `CallDescriptionMap`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CallDescriptionSet`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `CDM`: A type alias that simplifies use of a more complex underlying type. / 用于简化复杂底层类型使用方式的类型别名。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/StaticAnalyzer/Core/PathSensitive/CallEvent.h`, `llvm/ADT/ArrayRef.h`, `llvm/Support/Compiler.h`, `optional`, `vector`
- Forward declarations / 前向声明: `IdentifierInfo`
- Namespace context / 命名空间上下文: `clang`, `ento`
- Macro-style dependencies / 宏式依赖: None / 无

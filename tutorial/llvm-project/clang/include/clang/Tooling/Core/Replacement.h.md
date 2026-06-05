# Replacement.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Tooling/Core/Replacement.h`
- Repository: `llvm-project`
- Purpose (EN): Framework for clang refactoring tools.
- 用途（中文）: 该文件为 Tooling::Core 子系统中的 Replacement 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

```cpp
 1: //===- Replacement.h - Framework for clang refactoring tools ----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //  Classes supporting refactorings that span multiple translation units.
10: //  While single translation unit refactorings are supported via the Rewriter,
11: //  when refactoring multiple translation units changes must be stored in a
12: //  SourceManager independent form, duplicate changes need to be removed, and
13: //  all changes must be applied at once at the end of the refactoring so that
14: //  the code is always parseable.
15: //
16: //===----------------------------------------------------------------------===//
17: 
18: #ifndef LLVM_CLANG_TOOLING_CORE_REPLACEMENT_H
19: #define LLVM_CLANG_TOOLING_CORE_REPLACEMENT_H
20: 
21: #include "clang/Basic/LangOptions.h"
22: #include "clang/Basic/SourceLocation.h"
23: #include "llvm/ADT/StringRef.h"
24: #include "llvm/Support/Compiler.h"
25: #include "llvm/Support/Error.h"
26: #include "llvm/Support/raw_ostream.h"
27: #include <map>
28: #include <optional>
29: #include <set>
30: #include <string>
31: #include <system_error>
32: #include <utility>
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/Basic/LangOptions.h`, `clang/Basic/SourceLocation.h`, `llvm/ADT/StringRef.h` and 9 more.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/Basic/LangOptions.h`, `clang/Basic/SourceLocation.h`, `llvm/ADT/StringRef.h` 以及另外 9 项依赖。

### Lines 33-64

```cpp
33: #include <vector>
34: 
35: namespace clang {
36: 
37: class FileManager;
38: class Rewriter;
39: class SourceManager;
40: 
41: namespace tooling {
42: 
43: /// A source range independent of the \c SourceManager.
44: class Range {
45: public:
46:   Range() = default;
47:   Range(unsigned Offset, unsigned Length) : Offset(Offset), Length(Length) {}
48: 
49:   /// Accessors.
50:   /// @{
51:   unsigned getOffset() const { return Offset; }
52:   unsigned getLength() const { return Length; }
53:   /// @}
54: 
55:   /// \name Range Predicates
56:   /// @{
57:   /// Whether this range overlaps with \p RHS or not.
58:   bool overlapsWith(Range RHS) const {
59:     return Offset + Length > RHS.Offset && Offset < RHS.Offset + RHS.Length;
60:   }
61: 
62:   /// Whether this range contains \p RHS or not.
63:   bool contains(Range RHS) const {
64:     return RHS.Offset >= Offset &&
```
- EN: This block imports dependencies such as `vector`. It opens, closes, or documents namespace scope for `clang`, `tooling`. Key type declarations here include `FileManager`, `Rewriter`, `SourceManager`, `Range`.
- 中文: 这一块引入了 `vector` 等依赖。 它打开、关闭或说明了 `clang`, `tooling` 的命名空间作用域。 这里的重要类型声明包括 `FileManager`, `Rewriter`, `SourceManager`, `Range`。

### Lines 65-96

```cpp
65:            (RHS.Offset + RHS.Length) <= (Offset + Length);
66:   }
67: 
68:   /// Whether this range equals to \p RHS or not.
69:   bool operator==(const Range &RHS) const {
70:     return Offset == RHS.getOffset() && Length == RHS.getLength();
71:   }
72:   /// @}
73: 
74: private:
75:   unsigned Offset = 0;
76:   unsigned Length = 0;
77: };
78: 
79: /// A text replacement.
80: ///
81: /// Represents a SourceManager independent replacement of a range of text in a
82: /// specific file.
83: class Replacement {
84: public:
85:   /// Creates an invalid (not applicable) replacement.
86:   Replacement();
87: 
88:   /// Creates a replacement of the range [Offset, Offset+Length) in
89:   /// FilePath with ReplacementText.
90:   ///
91:   /// \param FilePath A source file accessible via a SourceManager.
92:   /// \param Offset The byte offset of the start of the range in the file.
93:   /// \param Length The length of the range in bytes.
94:   Replacement(StringRef FilePath, unsigned Offset, unsigned Length,
95:               StringRef ReplacementText);
96: 
```
- EN: Key type declarations here include `Replacement`. It exposes API surface such as `getOffset`, `Replacement`.
- 中文: 这里的重要类型声明包括 `Replacement`。 它暴露了 `getOffset`, `Replacement` 等接口。

### Lines 97-128

```cpp
 97:   /// Creates a Replacement of the range [Start, Start+Length) with
 98:   /// ReplacementText.
 99:   Replacement(const SourceManager &Sources, SourceLocation Start,
100:               unsigned Length, StringRef ReplacementText);
101: 
102:   /// Creates a Replacement of the given range with ReplacementText.
103:   Replacement(const SourceManager &Sources, const CharSourceRange &Range,
104:               StringRef ReplacementText,
105:               const LangOptions &LangOpts = LangOptions());
106: 
107:   /// Creates a Replacement of the node with ReplacementText.
108:   template <typename Node>
109:   Replacement(const SourceManager &Sources, const Node &NodeToReplace,
110:               StringRef ReplacementText,
111:               const LangOptions &LangOpts = LangOptions());
112: 
113:   /// Returns whether this replacement can be applied to a file.
114:   ///
115:   /// Only replacements that are in a valid file can be applied.
116:   bool isApplicable() const;
117: 
118:   /// Accessors.
119:   /// @{
120:   StringRef getFilePath() const { return FilePath; }
121:   unsigned getOffset() const { return ReplacementRange.getOffset(); }
122:   unsigned getLength() const { return ReplacementRange.getLength(); }
123:   StringRef getReplacementText() const { return ReplacementText; }
124:   /// @}
125: 
126:   /// Applies the replacement on the Rewriter.
127:   bool apply(Rewriter &Rewrite) const;
128: 
```
- EN: It exposes API surface such as `LangOptions`, `isApplicable`, `getFilePath`, `getOffset`.
- 中文: 它暴露了 `LangOptions`, `isApplicable`, `getFilePath`, `getOffset` 等接口。

### Lines 129-160

```cpp
129:   /// Returns a human readable string representation.
130:   std::string toString() const;
131: 
132: private:
133:   void setFromSourceLocation(const SourceManager &Sources, SourceLocation Start,
134:                              unsigned Length, StringRef ReplacementText);
135:   void setFromSourceRange(const SourceManager &Sources,
136:                           const CharSourceRange &Range,
137:                           StringRef ReplacementText,
138:                           const LangOptions &LangOpts);
139: 
140:   std::string FilePath;
141:   Range ReplacementRange;
142:   std::string ReplacementText;
143: };
144: 
145: enum class replacement_error {
146:   fail_to_apply = 0,
147:   wrong_file_path,
148:   overlap_conflict,
149:   insert_conflict,
150: };
151: 
152: /// Carries extra error information in replacement-related llvm::Error,
153: /// e.g. fail applying replacements and replacements conflict.
154: class ReplacementError : public llvm::ErrorInfo<ReplacementError> {
155: public:
156:   ReplacementError(replacement_error Err) : Err(Err) {}
157: 
158:   /// Constructs an error related to an existing replacement.
159:   ReplacementError(replacement_error Err, Replacement Existing)
160:       : Err(Err), ExistingReplacement(std::move(Existing)) {}
```
- EN: Key type declarations here include `replacement_error`, `ReplacementError`. It introduces enum-based state or option sets such as `replacement_error`. It exposes API surface such as `toString`, `ReplacementError`, `Err`.
- 中文: 这里的重要类型声明包括 `replacement_error`, `ReplacementError`。 它引入了 `replacement_error` 等基于枚举的状态或选项集合。 它暴露了 `toString`, `ReplacementError`, `Err` 等接口。

### Lines 161-192

```cpp
161: 
162:   /// Constructs an error related to a new replacement and an existing
163:   /// replacement in a set of replacements.
164:   ReplacementError(replacement_error Err, Replacement New, Replacement Existing)
165:       : Err(Err), NewReplacement(std::move(New)),
166:         ExistingReplacement(std::move(Existing)) {}
167: 
168:   std::string message() const override;
169: 
170:   void log(raw_ostream &OS) const override { OS << message(); }
171: 
172:   replacement_error get() const { return Err; }
173: 
174:   static char ID;
175: 
176:   const std::optional<Replacement> &getNewReplacement() const {
177:     return NewReplacement;
178:   }
179: 
180:   const std::optional<Replacement> &getExistingReplacement() const {
181:     return ExistingReplacement;
182:   }
183: 
184: private:
185:   // Users are not expected to use error_code.
186:   std::error_code convertToErrorCode() const override {
187:     return llvm::inconvertibleErrorCode();
188:   }
189: 
190:   replacement_error Err;
191: 
192:   // A new replacement, which is to expected be added into a set of
```
- EN: It exposes API surface such as `ExistingReplacement`, `message`, `get`, `getNewReplacement`.
- 中文: 它暴露了 `ExistingReplacement`, `message`, `get`, `getNewReplacement` 等接口。

### Lines 193-224

```cpp
193:   // replacements, that is causing problem.
194:   std::optional<Replacement> NewReplacement;
195: 
196:   // An existing replacement in a replacements set that is causing problem.
197:   std::optional<Replacement> ExistingReplacement;
198: };
199: 
200: /// Less-than operator between two Replacements.
201: bool operator<(const Replacement &LHS, const Replacement &RHS);
202: 
203: /// Equal-to operator between two Replacements.
204: bool operator==(const Replacement &LHS, const Replacement &RHS);
205: inline bool operator!=(const Replacement &LHS, const Replacement &RHS) {
206:   return !(LHS == RHS);
207: }
208: 
209: /// Maintains a set of replacements that are conflict-free.
210: /// Two replacements are considered conflicts if they overlap or have the same
211: /// offset (i.e. order-dependent).
212: class Replacements {
213: private:
214:   using ReplacementsImpl = std::set<Replacement>;
215: 
216: public:
217:   using const_iterator = ReplacementsImpl::const_iterator;
218:   using const_reverse_iterator = ReplacementsImpl::const_reverse_iterator;
219: 
220:   Replacements() = default;
221: 
222:   explicit Replacements(const Replacement &R) { Replaces.insert(R); }
223: 
224:   /// Adds a new replacement \p R to the current set of replacements.
```
- EN: Key type declarations here include `Replacements`. It defines convenient aliases such as `ReplacementsImpl`, `const_iterator`, `const_reverse_iterator`. It exposes API surface such as `operator<`, `Replacements`.
- 中文: 这里的重要类型声明包括 `Replacements`。 它定义了 `ReplacementsImpl`, `const_iterator`, `const_reverse_iterator` 等便捷别名。 它暴露了 `operator<`, `Replacements` 等接口。

### Lines 225-256

```cpp
225:   /// \p R must have the same file path as all existing replacements.
226:   /// Returns `success` if the replacement is successfully inserted; otherwise,
227:   /// it returns an llvm::Error, i.e. there is a conflict between R and the
228:   /// existing replacements (i.e. they are order-dependent) or R's file path is
229:   /// different from the filepath of existing replacements. Callers must
230:   /// explicitly check the Error returned, and the returned error can be
231:   /// converted to a string message with `llvm::toString()`. This prevents users
232:   /// from adding order-dependent replacements. To control the order in which
233:   /// order-dependent replacements are applied, use merge({R}) with R referring
234:   /// to the changed code after applying all existing replacements.
235:   /// Two replacements A and B are considered order-independent if applying them
236:   /// in either order produces the same result. Note that the range of the
237:   /// replacement that is applied later still refers to the original code.
238:   /// These include (but not restricted to) replacements that:
239:   ///   - don't overlap (being directly adjacent is fine) and
240:   ///   - are overlapping deletions.
241:   ///   - are insertions at the same offset and applying them in either order
242:   ///     has the same effect, i.e. X + Y = Y + X when inserting X and Y
243:   ///     respectively.
244:   ///   - are identical replacements, i.e. applying the same replacement twice
245:   ///     is equivalent to applying it once.
246:   /// Examples:
247:   /// 1. Replacement A(0, 0, "a") and B(0, 0, "aa") are order-independent since
248:   ///    applying them in either order gives replacement (0, 0, "aaa").
249:   ///    However, A(0, 0, "a") and B(0, 0, "b") are order-dependent since
250:   ///    applying A first gives (0, 0, "ab") while applying B first gives (B, A,
251:   ///    "ba").
252:   /// 2. Replacement A(0, 2, "123") and B(0, 2, "123") are order-independent
253:   ///    since applying them in either order gives (0, 2, "123").
254:   /// 3. Replacement A(0, 3, "123") and B(2, 3, "321") are order-independent
255:   ///    since either order gives (0, 5, "12321").
256:   /// 4. Replacement A(0, 3, "ab") and B(0, 3, "ab") are order-independent since
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 257-288

```cpp
257:   ///    applying the same replacement twice is equivalent to applying it once.
258:   /// Replacements with offset UINT_MAX are special - we do not detect conflicts
259:   /// for such replacements since users may add them intentionally as a special
260:   /// category of replacements.
261:   llvm::Error add(const Replacement &R);
262: 
263:   /// Merges \p Replaces into the current replacements. \p Replaces
264:   /// refers to code after applying the current replacements.
265:   [[nodiscard]] Replacements merge(const Replacements &Replaces) const;
266: 
267:   // Returns the affected ranges in the changed code.
268:   std::vector<Range> getAffectedRanges() const;
269: 
270:   // Returns the new offset in the code after replacements being applied.
271:   // Note that if there is an insertion at Offset in the current replacements,
272:   // \p Offset will be shifted to Offset + Length in inserted text.
273:   unsigned getShiftedCodePosition(unsigned Position) const;
274: 
275:   unsigned size() const { return Replaces.size(); }
276: 
277:   void clear() { Replaces.clear(); }
278: 
279:   bool empty() const { return Replaces.empty(); }
280: 
281:   const_iterator begin() const { return Replaces.begin(); }
282: 
283:   const_iterator end() const { return Replaces.end(); }
284: 
285:   const_reverse_iterator rbegin() const  { return Replaces.rbegin(); }
286: 
287:   const_reverse_iterator rend() const { return Replaces.rend(); }
288: 
```
- EN: It exposes API surface such as `add`, `merge`, `getAffectedRanges`, `getShiftedCodePosition`.
- 中文: 它暴露了 `add`, `merge`, `getAffectedRanges`, `getShiftedCodePosition` 等接口。

### Lines 289-320

```cpp
289:   bool operator==(const Replacements &RHS) const {
290:     return Replaces == RHS.Replaces;
291:   }
292: 
293: private:
294:   Replacements(const_iterator Begin, const_iterator End)
295:       : Replaces(Begin, End) {}
296: 
297:   // Returns `R` with new range that refers to code after `Replaces` being
298:   // applied.
299:   Replacement getReplacementInChangedCode(const Replacement &R) const;
300: 
301:   // Returns a set of replacements that is equivalent to the current
302:   // replacements by merging all adjacent replacements. Two sets of replacements
303:   // are considered equivalent if they have the same effect when they are
304:   // applied.
305:   Replacements getCanonicalReplacements() const;
306: 
307:   // If `R` and all existing replacements are order-independent, then merge it
308:   // with `Replaces` and returns the merged replacements; otherwise, returns an
309:   // error.
310:   llvm::Expected<Replacements>
311:   mergeIfOrderIndependent(const Replacement &R) const;
312: 
313:   ReplacementsImpl Replaces;
314: };
315: 
316: /// Apply all replacements in \p Replaces to the Rewriter \p Rewrite.
317: ///
318: /// Replacement applications happen independently of the success of
319: /// other applications.
320: ///
```
- EN: It exposes API surface such as `Replaces`, `getReplacementInChangedCode`, `getCanonicalReplacements`, `mergeIfOrderIndependent`.
- 中文: 它暴露了 `Replaces`, `getReplacementInChangedCode`, `getCanonicalReplacements`, `mergeIfOrderIndependent` 等接口。

### Lines 321-352

```cpp
321: /// \returns true if all replacements apply. false otherwise.
322: bool applyAllReplacements(const Replacements &Replaces, Rewriter &Rewrite);
323: 
324: /// Applies all replacements in \p Replaces to \p Code.
325: ///
326: /// This completely ignores the path stored in each replacement. If all
327: /// replacements are applied successfully, this returns the code with
328: /// replacements applied; otherwise, an llvm::Error carrying llvm::StringError
329: /// is returned (the Error message can be converted to string using
330: /// `llvm::toString()` and 'std::error_code` in the `Error` should be ignored).
331: llvm::Expected<std::string> applyAllReplacements(StringRef Code,
332:                                                  const Replacements &Replaces);
333: 
334: /// Collection of Replacements generated from a single translation unit.
335: struct TranslationUnitReplacements {
336:   /// Name of the main source for the translation unit.
337:   std::string MainSourceFile;
338: 
339:   std::vector<Replacement> Replacements;
340: };
341: 
342: /// Calculates the new ranges after \p Replaces are applied. These
343: /// include both the original \p Ranges and the affected ranges of \p Replaces
344: /// in the new code.
345: ///
346: /// \pre Replacements must be for the same file.
347: ///
348: /// \return The new ranges after \p Replaces are applied. The new ranges will be
349: /// sorted and non-overlapping.
350: std::vector<Range>
351: calculateRangesAfterReplacements(const Replacements &Replaces,
352:                                  const std::vector<Range> &Ranges);
```
- EN: Key type declarations here include `TranslationUnitReplacements`. It exposes API surface such as `applyAllReplacements`.
- 中文: 这里的重要类型声明包括 `TranslationUnitReplacements`。 它暴露了 `applyAllReplacements` 等接口。

### Lines 353-374

```cpp
353: 
354: /// If there are multiple <File, Replacements> pairs with the same file
355: /// entry, we only keep one pair and discard the rest.
356: /// If a file does not exist, its corresponding replacements will be ignored.
357: std::map<std::string, Replacements> groupReplacementsByFile(
358:     FileManager &FileMgr,
359:     const std::map<std::string, Replacements> &FileToReplaces);
360: 
361: template <typename Node>
362: Replacement::Replacement(const SourceManager &Sources,
363:                          const Node &NodeToReplace, StringRef ReplacementText,
364:                          const LangOptions &LangOpts) {
365:   const CharSourceRange Range =
366:       CharSourceRange::getTokenRange(NodeToReplace->getSourceRange());
367:   setFromSourceRange(Sources, Range, ReplacementText, LangOpts);
368: }
369: 
370: } // namespace tooling
371: 
372: } // namespace clang
373: 
374: #endif // LLVM_CLANG_TOOLING_CORE_REPLACEMENT_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `tooling`, `clang`. It exposes API surface such as `getTokenRange`, `setFromSourceRange`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `tooling`, `clang` 的命名空间作用域。 它暴露了 `getTokenRange`, `setFromSourceRange` 等接口。

## Key Concepts / 关键概念

- `FileManager`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Rewriter`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `SourceManager`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Range`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Replacement`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `replacement_error`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `ReplacementError`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Replacements`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/Basic/LangOptions.h`, `clang/Basic/SourceLocation.h`, `llvm/ADT/StringRef.h`, `llvm/Support/Compiler.h`, `llvm/Support/Error.h`, `llvm/Support/raw_ostream.h`, `map`, `optional`, `set`, `string`, `system_error`, `utility`, `vector`
- Forward declarations / 前向声明: `FileManager`, `Rewriter`, `SourceManager`
- Namespace context / 命名空间上下文: `clang`, `tooling`
- Macro-style dependencies / 宏式依赖: None / 无

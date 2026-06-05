# ParsedTemplate.h — Code Analysis / 代码分析

## Source / 来源

- File: `clang/include/clang/Sema/ParsedTemplate.h`
- Repository: `llvm-project`
- Purpose (EN): Template Parsing Data Types.
- 用途（中文）: 该文件为 Sema 子系统中的 Parsed Template 提供类型、接口或辅助声明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23

```cpp
 1: //===--- ParsedTemplate.h - Template Parsing Data Types ---------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: //  This file provides data structures that store the parsed representation of
10: //  templates.
11: //
12: //===----------------------------------------------------------------------===//
13: 
14: #ifndef LLVM_CLANG_SEMA_PARSEDTEMPLATE_H
15: #define LLVM_CLANG_SEMA_PARSEDTEMPLATE_H
16: 
17: #include "clang/Basic/OperatorKinds.h"
18: #include "clang/Basic/SourceLocation.h"
19: #include "clang/Basic/TemplateKinds.h"
20: #include "clang/Sema/DeclSpec.h"
21: #include "clang/Sema/Ownership.h"
22: #include "llvm/ADT/STLExtras.h"
23: #include "llvm/ADT/SmallVector.h"
```
- EN: This range establishes include guards or other file-scope compilation boundaries. This block imports dependencies such as `clang/Basic/OperatorKinds.h`, `clang/Basic/SourceLocation.h`, `clang/Basic/TemplateKinds.h` and 4 more.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 这一块引入了 `clang/Basic/OperatorKinds.h`, `clang/Basic/SourceLocation.h`, `clang/Basic/TemplateKinds.h` 以及另外 4 项依赖。

### Lines 24-46

```cpp
24: #include <cassert>
25: #include <cstdlib>
26: #include <new>
27: 
28: namespace clang {
29:   /// Represents the parsed form of a C++ template argument.
30:   class ParsedTemplateArgument {
31:   public:
32:     /// Describes the kind of template argument that was parsed.
33:     enum KindType {
34:       /// A template type parameter, stored as a type.
35:       Type,
36:       /// A non-type template parameter, stored as an expression.
37:       NonType,
38:       /// A template template argument, stored as a template name.
39:       Template
40:     };
41: 
42:     /// Build an empty template argument.
43:     ///
44:     /// This template argument is invalid.
45:     ParsedTemplateArgument() : Kind(Type), Arg(nullptr) { }
46: 
```
- EN: This block imports dependencies such as `cassert`, `cstdlib`, `new`. It opens, closes, or documents namespace scope for `clang`. Key type declarations here include `ParsedTemplateArgument`.
- 中文: 这一块引入了 `cassert`, `cstdlib`, `new` 等依赖。 它打开、关闭或说明了 `clang` 的命名空间作用域。 这里的重要类型声明包括 `ParsedTemplateArgument`。

### Lines 47-69

```cpp
47:     /// Create a template type argument or non-type template argument.
48:     ///
49:     /// \param Arg the template type argument or non-type template argument.
50:     /// \param Loc the location of the type.
51:     ParsedTemplateArgument(KindType Kind, void *Arg, SourceLocation NameLoc)
52:         : Kind(Kind), Arg(Arg), NameLoc(NameLoc) {}
53: 
54:     /// Create a template template argument.
55:     ///
56:     /// \param SS the C++ scope specifier that precedes the template name, if
57:     /// any.
58:     ///
59:     /// \param Template the template to which this template template
60:     /// argument refers.
61:     ///
62:     /// \param TemplateLoc the location of the template name.
63:     ParsedTemplateArgument(SourceLocation TemplateKwLoc, const CXXScopeSpec &SS,
64:                            ParsedTemplateTy Template, SourceLocation NameLoc)
65:         : Kind(ParsedTemplateArgument::Template),
66:           Arg(Template.getAsOpaquePtr()), SS(SS), TemplateKwLoc(TemplateKwLoc),
67:           NameLoc(NameLoc) {}
68: 
69:     /// Determine whether the given template argument is invalid.
```
- EN: It exposes API surface such as `Kind`, `NameLoc`.
- 中文: 它暴露了 `Kind`, `NameLoc` 等接口。

### Lines 70-92

```cpp
70:     bool isInvalid() const { return Arg == nullptr; }
71: 
72:     /// Determine what kind of template argument we have.
73:     KindType getKind() const { return Kind; }
74: 
75:     /// Retrieve the template type argument's type.
76:     ParsedType getAsType() const {
77:       assert(Kind == Type && "Not a template type argument");
78:       return ParsedType::getFromOpaquePtr(Arg);
79:     }
80: 
81:     /// Retrieve the non-type template argument's expression.
82:     Expr *getAsExpr() const {
83:       assert(Kind == NonType && "Not a non-type template argument");
84:       return static_cast<Expr*>(Arg);
85:     }
86: 
87:     /// Retrieve the template template argument's template name.
88:     ParsedTemplateTy getAsTemplate() const {
89:       assert(Kind == Template && "Not a template template argument");
90:       return ParsedTemplateTy::getFromOpaquePtr(Arg);
91:     }
92: 
```
- EN: It exposes API surface such as `isInvalid`, `getKind`, `getAsType`, `assert`.
- 中文: 它暴露了 `isInvalid`, `getKind`, `getAsType`, `assert` 等接口。

### Lines 93-115

```cpp
 93:     /// Retrieve the location of the template argument.
 94:     SourceLocation getTemplateKwLoc() const { return TemplateKwLoc; }
 95: 
 96:     /// Retrieve the location of the template argument.
 97:     SourceLocation getNameLoc() const { return NameLoc; }
 98: 
 99:     /// Retrieve the nested-name-specifier that precedes the template
100:     /// name in a template template argument.
101:     const CXXScopeSpec &getScopeSpec() const {
102:       assert(Kind == Template &&
103:              "Only template template arguments can have a scope specifier");
104:       return SS;
105:     }
106: 
107:     /// Retrieve the location of the ellipsis that makes a template
108:     /// template argument into a pack expansion.
109:     SourceLocation getEllipsisLoc() const {
110:       assert(Kind == Template &&
111:              "Only template template arguments can have an ellipsis");
112:       return EllipsisLoc;
113:     }
114: 
115:     /// Retrieve a pack expansion of the given template template
```
- EN: It exposes API surface such as `getTemplateKwLoc`, `getNameLoc`, `getScopeSpec`, `getEllipsisLoc`.
- 中文: 它暴露了 `getTemplateKwLoc`, `getNameLoc`, `getScopeSpec`, `getEllipsisLoc` 等接口。

### Lines 116-138

```cpp
116:     /// argument.
117:     ///
118:     /// \param EllipsisLoc The location of the ellipsis.
119:     ParsedTemplateArgument getTemplatePackExpansion(
120:                                               SourceLocation EllipsisLoc) const;
121: 
122:   private:
123:     KindType Kind;
124: 
125:     /// The actual template argument representation, which may be
126:     /// an \c Sema::TypeTy* (for a type), an Expr* (for an
127:     /// expression), or an Sema::TemplateTy (for a template).
128:     void *Arg;
129: 
130:     /// The nested-name-specifier that can accompany a template template
131:     /// argument.
132:     CXXScopeSpec SS;
133: 
134:     /// the location of the template keyword.
135:     SourceLocation TemplateKwLoc;
136: 
137:     /// the location of the template name.
138:     SourceLocation NameLoc;
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 139-161

```cpp
139: 
140:     /// The ellipsis location that can accompany a template template
141:     /// argument (turning it into a template template argument expansion).
142:     SourceLocation EllipsisLoc;
143:   };
144: 
145:   /// Information about a template-id annotation
146:   /// token.
147:   ///
148:   /// A template-id annotation token contains the template name,
149:   /// template arguments, and the source locations for important
150:   /// tokens. All of the information about template arguments is allocated
151:   /// directly after this structure.
152:   /// A template-id annotation token can also be generated by a type-constraint
153:   /// construct with no explicit template arguments, e.g. "template<C T>" would
154:   /// annotate C as a TemplateIdAnnotation with no template arguments (the angle
155:   /// locations would be invalid in this case).
156:   struct TemplateIdAnnotation final
157:       : private llvm::TrailingObjects<TemplateIdAnnotation,
158:                                       ParsedTemplateArgument> {
159:     friend TrailingObjects;
160:     /// TemplateKWLoc - The location of the template keyword.
161:     /// For e.g. typename T::template Y<U>
```
- EN: Key type declarations here include `TemplateIdAnnotation`.
- 中文: 这里的重要类型声明包括 `TemplateIdAnnotation`。

### Lines 162-184

```cpp
162:     SourceLocation TemplateKWLoc;
163: 
164:     /// TemplateNameLoc - The location of the template name within the
165:     /// source.
166:     SourceLocation TemplateNameLoc;
167: 
168:     /// FIXME: Temporarily stores the name of a specialization
169:     const IdentifierInfo *Name;
170: 
171:     /// FIXME: Temporarily stores the overloaded operator kind.
172:     OverloadedOperatorKind Operator;
173: 
174:     /// The declaration of the template corresponding to the
175:     /// template-name.
176:     ParsedTemplateTy Template;
177: 
178:     /// The kind of template that Template refers to. If this is
179:     /// TNK_Non_template, an error was encountered and diagnosed
180:     /// when parsing or looking up the template name.
181:     TemplateNameKind Kind;
182: 
183:     /// The location of the '<' before the template argument
184:     /// list.
```
- EN: The lines are primarily commentary that explains the surrounding design or declarations.
- 中文: 这些行主要是注释，用来解释周围的设计或声明。

### Lines 185-207

```cpp
185:     SourceLocation LAngleLoc;
186: 
187:     /// The location of the '>' after the template argument
188:     /// list.
189:     SourceLocation RAngleLoc;
190: 
191:     /// NumArgs - The number of template arguments.
192:     unsigned NumArgs;
193: 
194:     /// Whether an error was encountered in the template arguments.
195:     /// If so, NumArgs and the trailing arguments are best-effort.
196:     bool ArgsInvalid;
197: 
198:     /// Retrieves a pointer to the template arguments
199:     ParsedTemplateArgument *getTemplateArgs() { return getTrailingObjects(); }
200: 
201:     /// Creates a new TemplateIdAnnotation with NumArgs arguments and
202:     /// appends it to List.
203:     static TemplateIdAnnotation *
204:     Create(SourceLocation TemplateKWLoc, SourceLocation TemplateNameLoc,
205:            const IdentifierInfo *Name, OverloadedOperatorKind OperatorKind,
206:            ParsedTemplateTy OpaqueTemplateName, TemplateNameKind TemplateKind,
207:            SourceLocation LAngleLoc, SourceLocation RAngleLoc,
```
- EN: It exposes API surface such as `getTemplateArgs`.
- 中文: 它暴露了 `getTemplateArgs` 等接口。

### Lines 208-230

```cpp
208:            ArrayRef<ParsedTemplateArgument> TemplateArgs, bool ArgsInvalid,
209:            SmallVectorImpl<TemplateIdAnnotation *> &CleanupList) {
210:       TemplateIdAnnotation *TemplateId = new (llvm::safe_malloc(
211:           totalSizeToAlloc<ParsedTemplateArgument>(TemplateArgs.size())))
212:           TemplateIdAnnotation(TemplateKWLoc, TemplateNameLoc, Name,
213:                                OperatorKind, OpaqueTemplateName, TemplateKind,
214:                                LAngleLoc, RAngleLoc, TemplateArgs, ArgsInvalid);
215:       CleanupList.push_back(TemplateId);
216:       return TemplateId;
217:     }
218: 
219:     void Destroy() {
220:       for (ParsedTemplateArgument &A :
221:            llvm::make_range(getTemplateArgs(), getTemplateArgs() + NumArgs))
222:         A.~ParsedTemplateArgument();
223:       this->~TemplateIdAnnotation();
224:       free(this);
225:     }
226: 
227:     /// Determine whether this might be a type template.
228:     bool mightBeType() const {
229:       return Kind == TNK_Non_template ||
230:              Kind == TNK_Type_template ||
```
- EN: It exposes API surface such as `push_back`, `Destroy`, `~ParsedTemplateArgument`, `~TemplateIdAnnotation`.
- 中文: 它暴露了 `push_back`, `Destroy`, `~ParsedTemplateArgument`, `~TemplateIdAnnotation` 等接口。

### Lines 231-253

```cpp
231:              Kind == TNK_Dependent_template_name ||
232:              Kind == TNK_Undeclared_template;
233:     }
234: 
235:     bool hasInvalidName() const { return Kind == TNK_Non_template; }
236:     bool hasInvalidArgs() const { return ArgsInvalid; }
237: 
238:     bool isInvalid() const { return hasInvalidName() || hasInvalidArgs(); }
239: 
240:   private:
241:     TemplateIdAnnotation(const TemplateIdAnnotation &) = delete;
242: 
243:     TemplateIdAnnotation(SourceLocation TemplateKWLoc,
244:                          SourceLocation TemplateNameLoc,
245:                          const IdentifierInfo *Name,
246:                          OverloadedOperatorKind OperatorKind,
247:                          ParsedTemplateTy OpaqueTemplateName,
248:                          TemplateNameKind TemplateKind,
249:                          SourceLocation LAngleLoc, SourceLocation RAngleLoc,
250:                          ArrayRef<ParsedTemplateArgument> TemplateArgs,
251:                          bool ArgsInvalid) noexcept
252:         : TemplateKWLoc(TemplateKWLoc), TemplateNameLoc(TemplateNameLoc),
253:           Name(Name), Operator(OperatorKind), Template(OpaqueTemplateName),
```
- EN: It exposes API surface such as `hasInvalidName`, `hasInvalidArgs`, `isInvalid`, `TemplateIdAnnotation`.
- 中文: 它暴露了 `hasInvalidName`, `hasInvalidArgs`, `isInvalid`, `TemplateIdAnnotation` 等接口。

### Lines 254-267

```cpp
254:           Kind(TemplateKind), LAngleLoc(LAngleLoc), RAngleLoc(RAngleLoc),
255:           NumArgs(TemplateArgs.size()), ArgsInvalid(ArgsInvalid) {
256: 
257:       llvm::uninitialized_copy(TemplateArgs, getTemplateArgs());
258:     }
259:     ~TemplateIdAnnotation() = default;
260:   };
261: 
262:   /// Retrieves the range of the given template parameter lists.
263:   SourceRange getTemplateParamsRange(TemplateParameterList const *const *Params,
264:                                      unsigned NumParams);
265: } // end namespace clang
266: 
267: #endif // LLVM_CLANG_SEMA_PARSEDTEMPLATE_H
```
- EN: This range establishes include guards or other file-scope compilation boundaries. It opens, closes, or documents namespace scope for `clang`. It exposes API surface such as `NumArgs`, `uninitialized_copy`, `~TemplateIdAnnotation`.
- 中文: 这一段建立了头文件保护或其他文件级编译边界。 它打开、关闭或说明了 `clang` 的命名空间作用域。 它暴露了 `NumArgs`, `uninitialized_copy`, `~TemplateIdAnnotation` 等接口。

## Key Concepts / 关键概念

- `ParsedTemplateArgument`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `KindType`: An enumeration that captures a bounded set of modes, states, or categories. / 用于表达有限模式、状态或类别的枚举。
- `TemplateIdAnnotation`: A declared class/struct that shapes this file's public data model or API surface. / 用于构成本文件公开数据模型或接口表面的类/结构体声明。
- `Kind`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `NameLoc`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `isInvalid`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getKind`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。
- `getAsType`: A declared function or method that contributes behavior exposed by this file. / 为本文件暴露行为提供支持的函数或方法声明。

## Dependencies / 依赖关系

- Direct includes / 直接包含: `clang/Basic/OperatorKinds.h`, `clang/Basic/SourceLocation.h`, `clang/Basic/TemplateKinds.h`, `clang/Sema/DeclSpec.h`, `clang/Sema/Ownership.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallVector.h`, `cassert`, `cstdlib`, `new`
- Forward declarations / 前向声明: None / 无
- Namespace context / 命名空间上下文: `clang`
- Macro-style dependencies / 宏式依赖: None / 无

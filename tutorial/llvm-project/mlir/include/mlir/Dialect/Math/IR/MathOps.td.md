# MathOps.td — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `mlir/include/mlir/Dialect/Math/IR/MathOps.td` | `mlir/include/mlir/Dialect/Math/IR/MathOps.td` |
| Repository | `llvm/llvm-project` | `llvm/llvm-project` |
| Purpose | Base class for math dialect ops. Ops in this dialect have no side effects and. | 该文件的主要内容为：Base class for math dialect ops. Ops in this dialect have no side effects and。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

```tablegen
   1: //===- MathOps.td - Math op definitions --------------------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef MATH_OPS
  10: #define MATH_OPS
  11: 
  12: include "mlir/Dialect/Arith/IR/ArithBase.td"
```

- **EN:** Dependencies — This block gathers reused headers or TableGen sources before further declarations.
  **CN:** 依赖——这一段先收集可复用的头文件或 TableGen 源，再进入后续声明。
- **EN L1:** This comment states: “===- MathOps.td - Math op definitions --------------------*- tablegen -*-===”, documenting the intent of the surrounding code.
  **CN L1:** 该注释写道：“===- MathOps.td - Math op definitions --------------------*- tablegen -*-===”，用于说明周围代码的意图。
- **EN L2:** This comment documents context for the surrounding code.
  **CN L2:** 该注释为周围代码提供上下文说明。
- **EN L3:** This comment states: “Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.”, documenting the intent of the surrounding code.
  **CN L3:** 该注释写道：“Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.”，用于说明周围代码的意图。
- **EN L4:** This comment states: “See https://llvm.org/LICENSE.txt for license information.”, documenting the intent of the surrounding code.
  **CN L4:** 该注释写道：“See https://llvm.org/LICENSE.txt for license information.”，用于说明周围代码的意图。
- **EN L5:** This comment states: “SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception”, documenting the intent of the surrounding code.
  **CN L5:** 该注释写道：“SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception”，用于说明周围代码的意图。
- **EN L6:** This comment documents context for the surrounding code.
  **CN L6:** 该注释为周围代码提供上下文说明。
- **EN L7:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L7:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L8:** Blank line used to separate nearby declarations and improve readability.
  **CN L8:** 该空行用于分隔相邻声明并提升可读性。
- **EN L9:** This preprocessor directive manages `MATH_OPS` as part of the file's conditional compilation boundary.
  **CN L9:** 该预处理指令管理 `MATH_OPS`，作为文件条件编译边界的一部分。
- **EN L10:** This preprocessor directive manages `MATH_OPS` as part of the file's conditional compilation boundary.
  **CN L10:** 该预处理指令管理 `MATH_OPS`，作为文件条件编译边界的一部分。
- **EN L11:** Blank line used to separate nearby declarations and improve readability.
  **CN L11:** 该空行用于分隔相邻声明并提升可读性。
- **EN L12:** This TableGen include reuses records from `mlir/Dialect/Arith/IR/ArithBase.td`.
  **CN L12:** 该 TableGen include 复用了 `mlir/Dialect/Arith/IR/ArithBase.td` 中的记录。

### Lines 13-24 / 第 13-24 行

```tablegen
  13: include "mlir/Dialect/Arith/IR/ArithOpsInterfaces.td"
  14: include "mlir/Dialect/Math/IR/MathBase.td"
  15: include "mlir/Interfaces/InferTypeOpInterface.td"
  16: include "mlir/Interfaces/VectorInterfaces.td"
  17: include "mlir/Interfaces/SideEffectInterfaces.td"
  18: 
  19: // Base class for math dialect ops. Ops in this dialect have no side effects and
  20: // can be applied element-wise to vectors and tensors.
  21: class Math_Op<string mnemonic, list<Trait> traits = []> :
  22:     Op<Math_Dialect, mnemonic, traits # [Pure,
  23:     DeclareOpInterfaceMethods<VectorUnrollOpInterface>] #
  24:     ElementwiseMappable.traits>;
```

- **EN:** Dependencies — This block gathers reused headers or TableGen sources before further declarations.
  **CN:** 依赖——这一段先收集可复用的头文件或 TableGen 源，再进入后续声明。
- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L13:** This TableGen include reuses records from `mlir/Dialect/Arith/IR/ArithOpsInterfaces.td`.
  **CN L13:** 该 TableGen include 复用了 `mlir/Dialect/Arith/IR/ArithOpsInterfaces.td` 中的记录。
- **EN L14:** This TableGen include reuses records from `mlir/Dialect/Math/IR/MathBase.td`.
  **CN L14:** 该 TableGen include 复用了 `mlir/Dialect/Math/IR/MathBase.td` 中的记录。
- **EN L15:** This TableGen include reuses records from `mlir/Interfaces/InferTypeOpInterface.td`.
  **CN L15:** 该 TableGen include 复用了 `mlir/Interfaces/InferTypeOpInterface.td` 中的记录。
- **EN L16:** This TableGen include reuses records from `mlir/Interfaces/VectorInterfaces.td`.
  **CN L16:** 该 TableGen include 复用了 `mlir/Interfaces/VectorInterfaces.td` 中的记录。
- **EN L17:** This TableGen include reuses records from `mlir/Interfaces/SideEffectInterfaces.td`.
  **CN L17:** 该 TableGen include 复用了 `mlir/Interfaces/SideEffectInterfaces.td` 中的记录。
- **EN L18:** Blank line used to separate nearby declarations and improve readability.
  **CN L18:** 该空行用于分隔相邻声明并提升可读性。
- **EN L19:** This comment states: “Base class for math dialect ops. Ops in this dialect have no side effects and”, documenting the intent of the surrounding code.
  **CN L19:** 该注释写道：“Base class for math dialect ops. Ops in this dialect have no side effects and”，用于说明周围代码的意图。
- **EN L20:** This comment states: “can be applied element-wise to vectors and tensors.”, documenting the intent of the surrounding code.
  **CN L20:** 该注释写道：“can be applied element-wise to vectors and tensors.”，用于说明周围代码的意图。
- **EN L21:** This TableGen `class` record introduces `Math_Op`, which later participates in generated MLIR code.
  **CN L21:** 该 TableGen `class` 记录引入了 `Math_Op`，后续会参与生成的 MLIR 代码。
- **EN L22:** This line contributes implementation detail or declarative structure to the file.
  **CN L22:** 这一行为文件补充了实现细节或声明式结构。
- **EN L23:** This line contributes implementation detail or declarative structure to the file.
  **CN L23:** 这一行为文件补充了实现细节或声明式结构。
- **EN L24:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L24:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 25-36 / 第 25-36 行

```tablegen
  25: 
  26: // Base class for unary math operations on integer types. Require an operand
  27: // and result of the same type. This type can be an integer type, vector or
  28: // tensor thereof.
  29: class Math_IntegerUnaryOp<string mnemonic, list<Trait> traits = []> :
  30:     Math_Op<mnemonic, traits # [SameOperandsAndResultType]> {
  31:   let arguments = (ins SignlessIntegerOrIndexLike:$operand);
  32:   let results = (outs SignlessIntegerOrIndexLike:$result);
  33: 
  34:   let assemblyFormat = "$operand attr-dict `:` type($result)";
  35: }
  36: 
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L25:** Blank line used to separate nearby declarations and improve readability.
  **CN L25:** 该空行用于分隔相邻声明并提升可读性。
- **EN L26:** This comment states: “Base class for unary math operations on integer types. Require an operand”, documenting the intent of the surrounding code.
  **CN L26:** 该注释写道：“Base class for unary math operations on integer types. Require an operand”，用于说明周围代码的意图。
- **EN L27:** This comment states: “and result of the same type. This type can be an integer type, vector or”, documenting the intent of the surrounding code.
  **CN L27:** 该注释写道：“and result of the same type. This type can be an integer type, vector or”，用于说明周围代码的意图。
- **EN L28:** This comment states: “tensor thereof.”, documenting the intent of the surrounding code.
  **CN L28:** 该注释写道：“tensor thereof.”，用于说明周围代码的意图。
- **EN L29:** This TableGen `class` record introduces `Math_IntegerUnaryOp`, which later participates in generated MLIR code.
  **CN L29:** 该 TableGen `class` 记录引入了 `Math_IntegerUnaryOp`，后续会参与生成的 MLIR 代码。
- **EN L30:** This line contributes implementation detail or declarative structure to the file.
  **CN L30:** 这一行为文件补充了实现细节或声明式结构。
- **EN L31:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L31:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L32:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L32:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L33:** Blank line used to separate nearby declarations and improve readability.
  **CN L33:** 该空行用于分隔相邻声明并提升可读性。
- **EN L34:** This line contributes to the declaration or call of `type`.
  **CN L34:** 这一行为 `type` 的声明或调用提供内容。
- **EN L35:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L35:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L36:** Blank line used to separate nearby declarations and improve readability.
  **CN L36:** 该空行用于分隔相邻声明并提升可读性。

### Lines 37-48 / 第 37-48 行

```tablegen
  37: // Base class for floating point classification ops. Require an operand and
  38: // result of the same shape, which can be a floating point scalar, a vector or a
  39: // tensor thereof.
  40: class Math_FloatClassificationOp<string mnemonic, list<Trait> traits = []> :
  41:     Math_Op<mnemonic,
  42:       traits # [DeclareOpInterfaceMethods<ArithFastMathInterface>,
  43:                 TypesMatchWith<
  44:           "result type has i1 element type and same shape as operands",
  45:           "operand", "result", "::getI1SameShape($_self)">]> {
  46:   let arguments = (ins FloatLike:$operand,
  47:       DefaultValuedAttr<Arith_FastMathAttr,
  48:                         "::mlir::arith::FastMathFlags::none">:$fastmath);
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L37:** This comment states: “Base class for floating point classification ops. Require an operand and”, documenting the intent of the surrounding code.
  **CN L37:** 该注释写道：“Base class for floating point classification ops. Require an operand and”，用于说明周围代码的意图。
- **EN L38:** This comment states: “result of the same shape, which can be a floating point scalar, a vector or a”, documenting the intent of the surrounding code.
  **CN L38:** 该注释写道：“result of the same shape, which can be a floating point scalar, a vector or a”，用于说明周围代码的意图。
- **EN L39:** This comment states: “tensor thereof.”, documenting the intent of the surrounding code.
  **CN L39:** 该注释写道：“tensor thereof.”，用于说明周围代码的意图。
- **EN L40:** This TableGen `class` record introduces `Math_FloatClassificationOp`, which later participates in generated MLIR code.
  **CN L40:** 该 TableGen `class` 记录引入了 `Math_FloatClassificationOp`，后续会参与生成的 MLIR 代码。
- **EN L41:** This line contributes implementation detail or declarative structure to the file.
  **CN L41:** 这一行为文件补充了实现细节或声明式结构。
- **EN L42:** This line contributes implementation detail or declarative structure to the file.
  **CN L42:** 这一行为文件补充了实现细节或声明式结构。
- **EN L43:** This line contributes implementation detail or declarative structure to the file.
  **CN L43:** 这一行为文件补充了实现细节或声明式结构。
- **EN L44:** This line contributes implementation detail or declarative structure to the file.
  **CN L44:** 这一行为文件补充了实现细节或声明式结构。
- **EN L45:** This line contributes to the declaration or call of `getI1SameShape`.
  **CN L45:** 这一行为 `getI1SameShape` 的声明或调用提供内容。
- **EN L46:** This line contributes implementation detail or declarative structure to the file.
  **CN L46:** 这一行为文件补充了实现细节或声明式结构。
- **EN L47:** This line contributes implementation detail or declarative structure to the file.
  **CN L47:** 这一行为文件补充了实现细节或声明式结构。
- **EN L48:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L48:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 49-60 / 第 49-60 行

```tablegen
  49:   let results = (outs BoolLike:$result);
  50: 
  51:   let assemblyFormat = "$operand attr-dict `:` type($operand)";
  52: }
  53: 
  54: // Base class for unary math operations on floating point types. Require an
  55: // operand and result of the same type. This type can be a floating point type,
  56: // vector or tensor thereof.
  57: class Math_FloatUnaryOp<string mnemonic, list<Trait> traits = []> :
  58:     Math_Op<mnemonic,
  59:         traits # [SameOperandsAndResultType,
  60:                   DeclareOpInterfaceMethods<ArithFastMathInterface>]> {
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L49:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L49:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L50:** Blank line used to separate nearby declarations and improve readability.
  **CN L50:** 该空行用于分隔相邻声明并提升可读性。
- **EN L51:** This line contributes to the declaration or call of `type`.
  **CN L51:** 这一行为 `type` 的声明或调用提供内容。
- **EN L52:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L52:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L53:** Blank line used to separate nearby declarations and improve readability.
  **CN L53:** 该空行用于分隔相邻声明并提升可读性。
- **EN L54:** This comment states: “Base class for unary math operations on floating point types. Require an”, documenting the intent of the surrounding code.
  **CN L54:** 该注释写道：“Base class for unary math operations on floating point types. Require an”，用于说明周围代码的意图。
- **EN L55:** This comment states: “operand and result of the same type. This type can be a floating point type,”, documenting the intent of the surrounding code.
  **CN L55:** 该注释写道：“operand and result of the same type. This type can be a floating point type,”，用于说明周围代码的意图。
- **EN L56:** This comment states: “vector or tensor thereof.”, documenting the intent of the surrounding code.
  **CN L56:** 该注释写道：“vector or tensor thereof.”，用于说明周围代码的意图。
- **EN L57:** This TableGen `class` record introduces `Math_FloatUnaryOp`, which later participates in generated MLIR code.
  **CN L57:** 该 TableGen `class` 记录引入了 `Math_FloatUnaryOp`，后续会参与生成的 MLIR 代码。
- **EN L58:** This line contributes implementation detail or declarative structure to the file.
  **CN L58:** 这一行为文件补充了实现细节或声明式结构。
- **EN L59:** This line contributes implementation detail or declarative structure to the file.
  **CN L59:** 这一行为文件补充了实现细节或声明式结构。
- **EN L60:** This line contributes implementation detail or declarative structure to the file.
  **CN L60:** 这一行为文件补充了实现细节或声明式结构。

### Lines 61-72 / 第 61-72 行

```tablegen
  61:   let arguments = (ins FloatLike:$operand,
  62:       DefaultValuedAttr<Arith_FastMathAttr,
  63:                         "::mlir::arith::FastMathFlags::none">:$fastmath);
  64:   let results = (outs FloatLike:$result);
  65: 
  66:   let assemblyFormat = [{ $operand (`fastmath` `` $fastmath^)?
  67:                           attr-dict `:` type($result) }];
  68: }
  69: 
  70: // Base class for binary math operations on integer types. Require two
  71: // operands and one result of the same type. This type can be an integer
  72: // type, vector or tensor thereof.
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L61:** This line contributes implementation detail or declarative structure to the file.
  **CN L61:** 这一行为文件补充了实现细节或声明式结构。
- **EN L62:** This line contributes implementation detail or declarative structure to the file.
  **CN L62:** 这一行为文件补充了实现细节或声明式结构。
- **EN L63:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L63:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L64:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L64:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L65:** Blank line used to separate nearby declarations and improve readability.
  **CN L65:** 该空行用于分隔相邻声明并提升可读性。
- **EN L66:** This line contributes to the declaration or call of `operand`.
  **CN L66:** 这一行为 `operand` 的声明或调用提供内容。
- **EN L67:** This line contributes to the declaration or call of `type`.
  **CN L67:** 这一行为 `type` 的声明或调用提供内容。
- **EN L68:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L68:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L69:** Blank line used to separate nearby declarations and improve readability.
  **CN L69:** 该空行用于分隔相邻声明并提升可读性。
- **EN L70:** This comment states: “Base class for binary math operations on integer types. Require two”, documenting the intent of the surrounding code.
  **CN L70:** 该注释写道：“Base class for binary math operations on integer types. Require two”，用于说明周围代码的意图。
- **EN L71:** This comment states: “operands and one result of the same type. This type can be an integer”, documenting the intent of the surrounding code.
  **CN L71:** 该注释写道：“operands and one result of the same type. This type can be an integer”，用于说明周围代码的意图。
- **EN L72:** This comment states: “type, vector or tensor thereof.”, documenting the intent of the surrounding code.
  **CN L72:** 该注释写道：“type, vector or tensor thereof.”，用于说明周围代码的意图。

### Lines 73-84 / 第 73-84 行

```tablegen
  73: class Math_IntegerBinaryOp<string mnemonic, list<Trait> traits = []> :
  74:     Math_Op<mnemonic, traits # [SameOperandsAndResultType]> {
  75:   let arguments = (ins SignlessIntegerOrIndexLike:$lhs, SignlessIntegerOrIndexLike:$rhs);
  76:   let results = (outs SignlessIntegerOrIndexLike:$result);
  77: 
  78:   let assemblyFormat = "$lhs `,` $rhs attr-dict `:` type($result)";
  79: }
  80: 
  81: // Base class for binary math operations on floating point types. Require two
  82: // operands and one result of the same type. This type can be a floating point
  83: // type, vector or tensor thereof.
  84: class Math_FloatBinaryOp<string mnemonic, list<Trait> traits = []> :
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L73:** This TableGen `class` record introduces `Math_IntegerBinaryOp`, which later participates in generated MLIR code.
  **CN L73:** 该 TableGen `class` 记录引入了 `Math_IntegerBinaryOp`，后续会参与生成的 MLIR 代码。
- **EN L74:** This line contributes implementation detail or declarative structure to the file.
  **CN L74:** 这一行为文件补充了实现细节或声明式结构。
- **EN L75:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L75:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L76:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L76:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L77:** Blank line used to separate nearby declarations and improve readability.
  **CN L77:** 该空行用于分隔相邻声明并提升可读性。
- **EN L78:** This line contributes to the declaration or call of `type`.
  **CN L78:** 这一行为 `type` 的声明或调用提供内容。
- **EN L79:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L79:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L80:** Blank line used to separate nearby declarations and improve readability.
  **CN L80:** 该空行用于分隔相邻声明并提升可读性。
- **EN L81:** This comment states: “Base class for binary math operations on floating point types. Require two”, documenting the intent of the surrounding code.
  **CN L81:** 该注释写道：“Base class for binary math operations on floating point types. Require two”，用于说明周围代码的意图。
- **EN L82:** This comment states: “operands and one result of the same type. This type can be a floating point”, documenting the intent of the surrounding code.
  **CN L82:** 该注释写道：“operands and one result of the same type. This type can be a floating point”，用于说明周围代码的意图。
- **EN L83:** This comment states: “type, vector or tensor thereof.”, documenting the intent of the surrounding code.
  **CN L83:** 该注释写道：“type, vector or tensor thereof.”，用于说明周围代码的意图。
- **EN L84:** This TableGen `class` record introduces `Math_FloatBinaryOp`, which later participates in generated MLIR code.
  **CN L84:** 该 TableGen `class` 记录引入了 `Math_FloatBinaryOp`，后续会参与生成的 MLIR 代码。

### Lines 85-96 / 第 85-96 行

```tablegen
  85:     Math_Op<mnemonic,
  86:         traits # [SameOperandsAndResultType,
  87:                   DeclareOpInterfaceMethods<ArithFastMathInterface>]> {
  88:   let arguments = (ins FloatLike:$lhs, FloatLike:$rhs,
  89:       DefaultValuedAttr<Arith_FastMathAttr,
  90:                         "::mlir::arith::FastMathFlags::none">:$fastmath);
  91:   let results = (outs FloatLike:$result);
  92: 
  93:   let assemblyFormat = [{ $lhs `,` $rhs (`fastmath` `` $fastmath^)?
  94:                           attr-dict `:` type($result) }];
  95: }
  96: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L85:** This line contributes implementation detail or declarative structure to the file.
  **CN L85:** 这一行为文件补充了实现细节或声明式结构。
- **EN L86:** This line contributes implementation detail or declarative structure to the file.
  **CN L86:** 这一行为文件补充了实现细节或声明式结构。
- **EN L87:** This line contributes implementation detail or declarative structure to the file.
  **CN L87:** 这一行为文件补充了实现细节或声明式结构。
- **EN L88:** This line contributes implementation detail or declarative structure to the file.
  **CN L88:** 这一行为文件补充了实现细节或声明式结构。
- **EN L89:** This line contributes implementation detail or declarative structure to the file.
  **CN L89:** 这一行为文件补充了实现细节或声明式结构。
- **EN L90:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L90:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L91:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L91:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L92:** Blank line used to separate nearby declarations and improve readability.
  **CN L92:** 该空行用于分隔相邻声明并提升可读性。
- **EN L93:** This line contributes to the declaration or call of `rhs`.
  **CN L93:** 这一行为 `rhs` 的声明或调用提供内容。
- **EN L94:** This line contributes to the declaration or call of `type`.
  **CN L94:** 这一行为 `type` 的声明或调用提供内容。
- **EN L95:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L95:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L96:** Blank line used to separate nearby declarations and improve readability.
  **CN L96:** 该空行用于分隔相邻声明并提升可读性。

### Lines 97-108 / 第 97-108 行

```tablegen
  97: // Base class for floating point ternary operations. Require three operands and
  98: // one result of the same type. This type can be a floating point type, vector
  99: // or tensor thereof.
 100: class Math_FloatTernaryOp<string mnemonic, list<Trait> traits = []> :
 101:     Math_Op<mnemonic,
 102:         traits # [SameOperandsAndResultType,
 103:                   DeclareOpInterfaceMethods<ArithFastMathInterface>]> {
 104:   let arguments = (ins FloatLike:$a, FloatLike:$b, FloatLike:$c,
 105:       DefaultValuedAttr<Arith_FastMathAttr,
 106:                         "::mlir::arith::FastMathFlags::none">:$fastmath);
 107:   let results = (outs FloatLike:$result);
 108: 
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L97:** This comment states: “Base class for floating point ternary operations. Require three operands and”, documenting the intent of the surrounding code.
  **CN L97:** 该注释写道：“Base class for floating point ternary operations. Require three operands and”，用于说明周围代码的意图。
- **EN L98:** This comment states: “one result of the same type. This type can be a floating point type, vector”, documenting the intent of the surrounding code.
  **CN L98:** 该注释写道：“one result of the same type. This type can be a floating point type, vector”，用于说明周围代码的意图。
- **EN L99:** This comment states: “or tensor thereof.”, documenting the intent of the surrounding code.
  **CN L99:** 该注释写道：“or tensor thereof.”，用于说明周围代码的意图。
- **EN L100:** This TableGen `class` record introduces `Math_FloatTernaryOp`, which later participates in generated MLIR code.
  **CN L100:** 该 TableGen `class` 记录引入了 `Math_FloatTernaryOp`，后续会参与生成的 MLIR 代码。
- **EN L101:** This line contributes implementation detail or declarative structure to the file.
  **CN L101:** 这一行为文件补充了实现细节或声明式结构。
- **EN L102:** This line contributes implementation detail or declarative structure to the file.
  **CN L102:** 这一行为文件补充了实现细节或声明式结构。
- **EN L103:** This line contributes implementation detail or declarative structure to the file.
  **CN L103:** 这一行为文件补充了实现细节或声明式结构。
- **EN L104:** This line contributes implementation detail or declarative structure to the file.
  **CN L104:** 这一行为文件补充了实现细节或声明式结构。
- **EN L105:** This line contributes implementation detail or declarative structure to the file.
  **CN L105:** 这一行为文件补充了实现细节或声明式结构。
- **EN L106:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L106:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L107:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L107:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L108:** Blank line used to separate nearby declarations and improve readability.
  **CN L108:** 该空行用于分隔相邻声明并提升可读性。

### Lines 109-120 / 第 109-120 行

```tablegen
 109:   let assemblyFormat = [{ $a `,` $b `,` $c (`fastmath` `` $fastmath^)?
 110:                           attr-dict `:` type($result) }];
 111: }
 112: 
 113: // Base class for floating point ternary operations with an optional rounding
 114: // mode.
 115: class Math_FloatTernaryOpWithRoundingMode<string mnemonic,
 116:                                           list<Trait> traits = []> :
 117:     Math_FloatTernaryOp<mnemonic,
 118:       !listconcat([DeclareOpInterfaceMethods<ArithRoundingModeInterface>],
 119:                   traits)> {
 120:   let arguments = (ins FloatLike:$a, FloatLike:$b, FloatLike:$c,
```

- **EN:** Symbols — This block introduces named types or enumerations that define the file interface.
  **CN:** 符号——这一段引入具名类型或枚举，用于定义文件接口。
- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L109:** This line contributes to the declaration or call of `c`.
  **CN L109:** 这一行为 `c` 的声明或调用提供内容。
- **EN L110:** This line contributes to the declaration or call of `type`.
  **CN L110:** 这一行为 `type` 的声明或调用提供内容。
- **EN L111:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L111:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L112:** Blank line used to separate nearby declarations and improve readability.
  **CN L112:** 该空行用于分隔相邻声明并提升可读性。
- **EN L113:** This comment states: “Base class for floating point ternary operations with an optional rounding”, documenting the intent of the surrounding code.
  **CN L113:** 该注释写道：“Base class for floating point ternary operations with an optional rounding”，用于说明周围代码的意图。
- **EN L114:** This comment states: “mode.”, documenting the intent of the surrounding code.
  **CN L114:** 该注释写道：“mode.”，用于说明周围代码的意图。
- **EN L115:** This TableGen `class` record introduces `Math_FloatTernaryOpWithRoundingMode`, which later participates in generated MLIR code.
  **CN L115:** 该 TableGen `class` 记录引入了 `Math_FloatTernaryOpWithRoundingMode`，后续会参与生成的 MLIR 代码。
- **EN L116:** This line contributes implementation detail or declarative structure to the file.
  **CN L116:** 这一行为文件补充了实现细节或声明式结构。
- **EN L117:** This line contributes implementation detail or declarative structure to the file.
  **CN L117:** 这一行为文件补充了实现细节或声明式结构。
- **EN L118:** This line contributes to the declaration or call of `listconcat`.
  **CN L118:** 这一行为 `listconcat` 的声明或调用提供内容。
- **EN L119:** This line contributes implementation detail or declarative structure to the file.
  **CN L119:** 这一行为文件补充了实现细节或声明式结构。
- **EN L120:** This line contributes implementation detail or declarative structure to the file.
  **CN L120:** 这一行为文件补充了实现细节或声明式结构。

### Lines 121-132 / 第 121-132 行

```tablegen
 121:       DefaultValuedAttr<Arith_FastMathAttr,
 122:                         "::mlir::arith::FastMathFlags::none">:$fastmath,
 123:       OptionalAttr<Arith_RoundingModeAttr>:$roundingmode);
 124:   let builders = [
 125:     OpBuilder<(ins "Value":$a, "Value":$b, "Value":$c,
 126:       CArg<"::mlir::arith::FastMathFlags",
 127:            "::mlir::arith::FastMathFlags::none">:$fastmath), [{
 128:       build($_builder, $_state, a, b, c, fastmath,
 129:             ::mlir::arith::RoundingModeAttr{});
 130:     }]>,
 131:     OpBuilder<(ins "Value":$a, "Value":$b, "Value":$c,
 132:       "::mlir::arith::FastMathFlagsAttr":$fastmath), [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L121:** This line contributes implementation detail or declarative structure to the file.
  **CN L121:** 这一行为文件补充了实现细节或声明式结构。
- **EN L122:** This line contributes implementation detail or declarative structure to the file.
  **CN L122:** 这一行为文件补充了实现细节或声明式结构。
- **EN L123:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L123:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L124:** This line contributes implementation detail or declarative structure to the file.
  **CN L124:** 这一行为文件补充了实现细节或声明式结构。
- **EN L125:** This line contributes implementation detail or declarative structure to the file.
  **CN L125:** 这一行为文件补充了实现细节或声明式结构。
- **EN L126:** This line contributes implementation detail or declarative structure to the file.
  **CN L126:** 这一行为文件补充了实现细节或声明式结构。
- **EN L127:** This line contributes implementation detail or declarative structure to the file.
  **CN L127:** 这一行为文件补充了实现细节或声明式结构。
- **EN L128:** This line contributes to the declaration or call of `build`.
  **CN L128:** 这一行为 `build` 的声明或调用提供内容。
- **EN L129:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L129:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L130:** This line contributes implementation detail or declarative structure to the file.
  **CN L130:** 这一行为文件补充了实现细节或声明式结构。
- **EN L131:** This line contributes implementation detail or declarative structure to the file.
  **CN L131:** 这一行为文件补充了实现细节或声明式结构。
- **EN L132:** This line contributes implementation detail or declarative structure to the file.
  **CN L132:** 这一行为文件补充了实现细节或声明式结构。

### Lines 133-144 / 第 133-144 行

```tablegen
 133:       build($_builder, $_state, a, b, c, fastmath,
 134:             ::mlir::arith::RoundingModeAttr{});
 135:     }]>,
 136:     OpBuilder<(ins "Type":$type, "Value":$a, "Value":$b, "Value":$c,
 137:       CArg<"::mlir::arith::FastMathFlags",
 138:            "::mlir::arith::FastMathFlags::none">:$fastmath), [{
 139:       build($_builder, $_state, type, a, b, c,
 140:             ::mlir::arith::FastMathFlagsAttr::get(
 141:                 $_builder.getContext(), fastmath),
 142:             ::mlir::arith::RoundingModeAttr{});
 143:     }]>,
 144:   ];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L133:** This line contributes to the declaration or call of `build`.
  **CN L133:** 这一行为 `build` 的声明或调用提供内容。
- **EN L134:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L134:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L135:** This line contributes implementation detail or declarative structure to the file.
  **CN L135:** 这一行为文件补充了实现细节或声明式结构。
- **EN L136:** This line contributes implementation detail or declarative structure to the file.
  **CN L136:** 这一行为文件补充了实现细节或声明式结构。
- **EN L137:** This line contributes implementation detail or declarative structure to the file.
  **CN L137:** 这一行为文件补充了实现细节或声明式结构。
- **EN L138:** This line contributes implementation detail or declarative structure to the file.
  **CN L138:** 这一行为文件补充了实现细节或声明式结构。
- **EN L139:** This line contributes to the declaration or call of `build`.
  **CN L139:** 这一行为 `build` 的声明或调用提供内容。
- **EN L140:** This line contributes to the declaration or call of `get`.
  **CN L140:** 这一行为 `get` 的声明或调用提供内容。
- **EN L141:** This line contributes to the declaration or call of `getContext`.
  **CN L141:** 这一行为 `getContext` 的声明或调用提供内容。
- **EN L142:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L142:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L143:** This line contributes implementation detail or declarative structure to the file.
  **CN L143:** 这一行为文件补充了实现细节或声明式结构。
- **EN L144:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L144:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 145-156 / 第 145-156 行

```tablegen
 145:   let assemblyFormat = [{ $a `,` $b `,` $c ($roundingmode^)?
 146:                           (`fastmath` `` $fastmath^)?
 147:                           attr-dict `:` type($result) }];
 148: }
 149: 
 150: //===----------------------------------------------------------------------===//
 151: // AbsFOp
 152: //===----------------------------------------------------------------------===//
 153: 
 154: def Math_AbsFOp : Math_FloatUnaryOp<"absf"> {
 155:   let summary = "floating point absolute-value operation";
 156:   let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L145:** This line contributes to the declaration or call of `c`.
  **CN L145:** 这一行为 `c` 的声明或调用提供内容。
- **EN L146:** This line contributes implementation detail or declarative structure to the file.
  **CN L146:** 这一行为文件补充了实现细节或声明式结构。
- **EN L147:** This line contributes to the declaration or call of `type`.
  **CN L147:** 这一行为 `type` 的声明或调用提供内容。
- **EN L148:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L148:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L149:** Blank line used to separate nearby declarations and improve readability.
  **CN L149:** 该空行用于分隔相邻声明并提升可读性。
- **EN L150:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L150:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L151:** This comment states: “AbsFOp”, documenting the intent of the surrounding code.
  **CN L151:** 该注释写道：“AbsFOp”，用于说明周围代码的意图。
- **EN L152:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L152:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L153:** Blank line used to separate nearby declarations and improve readability.
  **CN L153:** 该空行用于分隔相邻声明并提升可读性。
- **EN L154:** This TableGen `def` record introduces `Math_AbsFOp`, which later participates in generated MLIR code.
  **CN L154:** 该 TableGen `def` 记录引入了 `Math_AbsFOp`，后续会参与生成的 MLIR 代码。
- **EN L155:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L155:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L156:** This line contributes implementation detail or declarative structure to the file.
  **CN L156:** 这一行为文件补充了实现细节或声明式结构。

### Lines 157-168 / 第 157-168 行

```tablegen
 157:     The `absf` operation computes the absolute value. It takes one operand of
 158:     floating point type (i.e., scalar, tensor or vector) and returns one result
 159:     of the same type.
 160: 
 161:     Example:
 162: 
 163:     ```mlir
 164:     // Scalar absolute value.
 165:     %a = math.absf %b : f64
 166:     ```
 167:   }];
 168:   let hasFolder = 1;
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L157:** This line contributes implementation detail or declarative structure to the file.
  **CN L157:** 这一行为文件补充了实现细节或声明式结构。
- **EN L158:** This line contributes to the declaration or call of `type`.
  **CN L158:** 这一行为 `type` 的声明或调用提供内容。
- **EN L159:** This line contributes implementation detail or declarative structure to the file.
  **CN L159:** 这一行为文件补充了实现细节或声明式结构。
- **EN L160:** Blank line used to separate nearby declarations and improve readability.
  **CN L160:** 该空行用于分隔相邻声明并提升可读性。
- **EN L161:** This line contributes implementation detail or declarative structure to the file.
  **CN L161:** 这一行为文件补充了实现细节或声明式结构。
- **EN L162:** Blank line used to separate nearby declarations and improve readability.
  **CN L162:** 该空行用于分隔相邻声明并提升可读性。
- **EN L163:** This line contributes implementation detail or declarative structure to the file.
  **CN L163:** 这一行为文件补充了实现细节或声明式结构。
- **EN L164:** This comment states: “Scalar absolute value.”, documenting the intent of the surrounding code.
  **CN L164:** 该注释写道：“Scalar absolute value.”，用于说明周围代码的意图。
- **EN L165:** This line contributes implementation detail or declarative structure to the file.
  **CN L165:** 这一行为文件补充了实现细节或声明式结构。
- **EN L166:** This line contributes implementation detail or declarative structure to the file.
  **CN L166:** 这一行为文件补充了实现细节或声明式结构。
- **EN L167:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L167:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L168:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L168:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 169-180 / 第 169-180 行

```tablegen
 169: }
 170: 
 171: //===----------------------------------------------------------------------===//
 172: // AbsIOp
 173: //===----------------------------------------------------------------------===//
 174: 
 175: def Math_AbsIOp : Math_IntegerUnaryOp<"absi"> {
 176:   let summary = "integer absolute-value operation";
 177:   let description = [{
 178:     The `absi` operation computes the absolute value. It takes one operand of
 179:     integer type (i.e., scalar, tensor or vector) and returns one result of the
 180:     same type.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L169:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L169:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L170:** Blank line used to separate nearby declarations and improve readability.
  **CN L170:** 该空行用于分隔相邻声明并提升可读性。
- **EN L171:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L171:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L172:** This comment states: “AbsIOp”, documenting the intent of the surrounding code.
  **CN L172:** 该注释写道：“AbsIOp”，用于说明周围代码的意图。
- **EN L173:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L173:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L174:** Blank line used to separate nearby declarations and improve readability.
  **CN L174:** 该空行用于分隔相邻声明并提升可读性。
- **EN L175:** This TableGen `def` record introduces `Math_AbsIOp`, which later participates in generated MLIR code.
  **CN L175:** 该 TableGen `def` 记录引入了 `Math_AbsIOp`，后续会参与生成的 MLIR 代码。
- **EN L176:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L176:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L177:** This line contributes implementation detail or declarative structure to the file.
  **CN L177:** 这一行为文件补充了实现细节或声明式结构。
- **EN L178:** This line contributes implementation detail or declarative structure to the file.
  **CN L178:** 这一行为文件补充了实现细节或声明式结构。
- **EN L179:** This line contributes to the declaration or call of `type`.
  **CN L179:** 这一行为 `type` 的声明或调用提供内容。
- **EN L180:** This line contributes implementation detail or declarative structure to the file.
  **CN L180:** 这一行为文件补充了实现细节或声明式结构。

### Lines 181-192 / 第 181-192 行

```tablegen
 181: 
 182:     Example:
 183: 
 184:     ```mlir
 185:     // Scalar absolute value.
 186:     %a = math.absi %b : i64
 187:     ```
 188:   }];
 189:   let hasFolder = 1;
 190: }
 191: 
 192: //===----------------------------------------------------------------------===//
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L181:** Blank line used to separate nearby declarations and improve readability.
  **CN L181:** 该空行用于分隔相邻声明并提升可读性。
- **EN L182:** This line contributes implementation detail or declarative structure to the file.
  **CN L182:** 这一行为文件补充了实现细节或声明式结构。
- **EN L183:** Blank line used to separate nearby declarations and improve readability.
  **CN L183:** 该空行用于分隔相邻声明并提升可读性。
- **EN L184:** This line contributes implementation detail or declarative structure to the file.
  **CN L184:** 这一行为文件补充了实现细节或声明式结构。
- **EN L185:** This comment states: “Scalar absolute value.”, documenting the intent of the surrounding code.
  **CN L185:** 该注释写道：“Scalar absolute value.”，用于说明周围代码的意图。
- **EN L186:** This line contributes implementation detail or declarative structure to the file.
  **CN L186:** 这一行为文件补充了实现细节或声明式结构。
- **EN L187:** This line contributes implementation detail or declarative structure to the file.
  **CN L187:** 这一行为文件补充了实现细节或声明式结构。
- **EN L188:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L188:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L189:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L189:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L190:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L190:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L191:** Blank line used to separate nearby declarations and improve readability.
  **CN L191:** 该空行用于分隔相邻声明并提升可读性。
- **EN L192:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L192:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 193-204 / 第 193-204 行

```tablegen
 193: // AcoshOp
 194: //===----------------------------------------------------------------------===//
 195: 
 196: def Math_AcoshOp : Math_FloatUnaryOp<"acosh">{
 197:   let summary = "Hyperbolic arcus cosine of the given value";
 198:   let description = [{
 199:     Syntax:
 200: 
 201:     ```
 202:     operation ::= ssa-id `=` `math.acosh` ssa-use `:` type
 203:     ```
 204: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L193:** This comment states: “AcoshOp”, documenting the intent of the surrounding code.
  **CN L193:** 该注释写道：“AcoshOp”，用于说明周围代码的意图。
- **EN L194:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L194:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L195:** Blank line used to separate nearby declarations and improve readability.
  **CN L195:** 该空行用于分隔相邻声明并提升可读性。
- **EN L196:** This TableGen `def` record introduces `Math_AcoshOp`, which later participates in generated MLIR code.
  **CN L196:** 该 TableGen `def` 记录引入了 `Math_AcoshOp`，后续会参与生成的 MLIR 代码。
- **EN L197:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L197:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L198:** This line contributes implementation detail or declarative structure to the file.
  **CN L198:** 这一行为文件补充了实现细节或声明式结构。
- **EN L199:** This line contributes implementation detail or declarative structure to the file.
  **CN L199:** 这一行为文件补充了实现细节或声明式结构。
- **EN L200:** Blank line used to separate nearby declarations and improve readability.
  **CN L200:** 该空行用于分隔相邻声明并提升可读性。
- **EN L201:** This line contributes implementation detail or declarative structure to the file.
  **CN L201:** 这一行为文件补充了实现细节或声明式结构。
- **EN L202:** This line contributes implementation detail or declarative structure to the file.
  **CN L202:** 这一行为文件补充了实现细节或声明式结构。
- **EN L203:** This line contributes implementation detail or declarative structure to the file.
  **CN L203:** 这一行为文件补充了实现细节或声明式结构。
- **EN L204:** Blank line used to separate nearby declarations and improve readability.
  **CN L204:** 该空行用于分隔相邻声明并提升可读性。

### Lines 205-216 / 第 205-216 行

```tablegen
 205:     The `acosh` operation computes the arcus cosine of a given value.  It takes
 206:     one operand of floating point type (i.e., scalar, tensor or vector) and returns
 207:     one result of the same type. It has no standard attributes.
 208: 
 209:     Example:
 210: 
 211:     ```mlir
 212:     // Hyperbolic arcus cosine of scalar value.
 213:     %a = math.acosh %b : f64
 214:     ```
 215:   }];
 216:   let hasFolder = 1;
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L205:** This line contributes implementation detail or declarative structure to the file.
  **CN L205:** 这一行为文件补充了实现细节或声明式结构。
- **EN L206:** This line contributes to the declaration or call of `type`.
  **CN L206:** 这一行为 `type` 的声明或调用提供内容。
- **EN L207:** This line contributes implementation detail or declarative structure to the file.
  **CN L207:** 这一行为文件补充了实现细节或声明式结构。
- **EN L208:** Blank line used to separate nearby declarations and improve readability.
  **CN L208:** 该空行用于分隔相邻声明并提升可读性。
- **EN L209:** This line contributes implementation detail or declarative structure to the file.
  **CN L209:** 这一行为文件补充了实现细节或声明式结构。
- **EN L210:** Blank line used to separate nearby declarations and improve readability.
  **CN L210:** 该空行用于分隔相邻声明并提升可读性。
- **EN L211:** This line contributes implementation detail or declarative structure to the file.
  **CN L211:** 这一行为文件补充了实现细节或声明式结构。
- **EN L212:** This comment states: “Hyperbolic arcus cosine of scalar value.”, documenting the intent of the surrounding code.
  **CN L212:** 该注释写道：“Hyperbolic arcus cosine of scalar value.”，用于说明周围代码的意图。
- **EN L213:** This line contributes implementation detail or declarative structure to the file.
  **CN L213:** 这一行为文件补充了实现细节或声明式结构。
- **EN L214:** This line contributes implementation detail or declarative structure to the file.
  **CN L214:** 这一行为文件补充了实现细节或声明式结构。
- **EN L215:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L215:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L216:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L216:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 217-228 / 第 217-228 行

```tablegen
 217: }
 218: 
 219: //===----------------------------------------------------------------------===//
 220: // AsinOp
 221: //===----------------------------------------------------------------------===//
 222: 
 223: def Math_AsinOp : Math_FloatUnaryOp<"asin">{
 224:   let summary = "arcus sine of the given value";
 225:   let description = [{
 226:     Syntax:
 227: 
 228:     ```
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L217:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L217:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L218:** Blank line used to separate nearby declarations and improve readability.
  **CN L218:** 该空行用于分隔相邻声明并提升可读性。
- **EN L219:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L219:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L220:** This comment states: “AsinOp”, documenting the intent of the surrounding code.
  **CN L220:** 该注释写道：“AsinOp”，用于说明周围代码的意图。
- **EN L221:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L221:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L222:** Blank line used to separate nearby declarations and improve readability.
  **CN L222:** 该空行用于分隔相邻声明并提升可读性。
- **EN L223:** This TableGen `def` record introduces `Math_AsinOp`, which later participates in generated MLIR code.
  **CN L223:** 该 TableGen `def` 记录引入了 `Math_AsinOp`，后续会参与生成的 MLIR 代码。
- **EN L224:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L224:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L225:** This line contributes implementation detail or declarative structure to the file.
  **CN L225:** 这一行为文件补充了实现细节或声明式结构。
- **EN L226:** This line contributes implementation detail or declarative structure to the file.
  **CN L226:** 这一行为文件补充了实现细节或声明式结构。
- **EN L227:** Blank line used to separate nearby declarations and improve readability.
  **CN L227:** 该空行用于分隔相邻声明并提升可读性。
- **EN L228:** This line contributes implementation detail or declarative structure to the file.
  **CN L228:** 这一行为文件补充了实现细节或声明式结构。

### Lines 229-240 / 第 229-240 行

```tablegen
 229:     operation ::= ssa-id `=` `math.asin` ssa-use `:` type
 230:     ```
 231: 
 232:     The `asin` operation computes the arcus sine of a given value.  It takes
 233:     one operand of floating point type (i.e., scalar, tensor or vector) and returns
 234:     one result of the same type. It has no standard attributes.
 235: 
 236:     Example:
 237: 
 238:     ```mlir
 239:     // Arcus sine of scalar value.
 240:     %a = math.asin %b : f64
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L229:** This line contributes implementation detail or declarative structure to the file.
  **CN L229:** 这一行为文件补充了实现细节或声明式结构。
- **EN L230:** This line contributes implementation detail or declarative structure to the file.
  **CN L230:** 这一行为文件补充了实现细节或声明式结构。
- **EN L231:** Blank line used to separate nearby declarations and improve readability.
  **CN L231:** 该空行用于分隔相邻声明并提升可读性。
- **EN L232:** This line contributes implementation detail or declarative structure to the file.
  **CN L232:** 这一行为文件补充了实现细节或声明式结构。
- **EN L233:** This line contributes to the declaration or call of `type`.
  **CN L233:** 这一行为 `type` 的声明或调用提供内容。
- **EN L234:** This line contributes implementation detail or declarative structure to the file.
  **CN L234:** 这一行为文件补充了实现细节或声明式结构。
- **EN L235:** Blank line used to separate nearby declarations and improve readability.
  **CN L235:** 该空行用于分隔相邻声明并提升可读性。
- **EN L236:** This line contributes implementation detail or declarative structure to the file.
  **CN L236:** 这一行为文件补充了实现细节或声明式结构。
- **EN L237:** Blank line used to separate nearby declarations and improve readability.
  **CN L237:** 该空行用于分隔相邻声明并提升可读性。
- **EN L238:** This line contributes implementation detail or declarative structure to the file.
  **CN L238:** 这一行为文件补充了实现细节或声明式结构。
- **EN L239:** This comment states: “Arcus sine of scalar value.”, documenting the intent of the surrounding code.
  **CN L239:** 该注释写道：“Arcus sine of scalar value.”，用于说明周围代码的意图。
- **EN L240:** This line contributes implementation detail or declarative structure to the file.
  **CN L240:** 这一行为文件补充了实现细节或声明式结构。

### Lines 241-252 / 第 241-252 行

```tablegen
 241:     ```
 242:   }];
 243:   let hasFolder = 1;
 244: }
 245: 
 246: //===----------------------------------------------------------------------===//
 247: // AsinhOp
 248: //===----------------------------------------------------------------------===//
 249: 
 250: def Math_AsinhOp : Math_FloatUnaryOp<"asinh">{
 251:   let summary = "hyperbolic arcus sine of the given value";
 252:   let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L241:** This line contributes implementation detail or declarative structure to the file.
  **CN L241:** 这一行为文件补充了实现细节或声明式结构。
- **EN L242:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L242:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L243:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L243:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L244:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L244:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L245:** Blank line used to separate nearby declarations and improve readability.
  **CN L245:** 该空行用于分隔相邻声明并提升可读性。
- **EN L246:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L246:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L247:** This comment states: “AsinhOp”, documenting the intent of the surrounding code.
  **CN L247:** 该注释写道：“AsinhOp”，用于说明周围代码的意图。
- **EN L248:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L248:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L249:** Blank line used to separate nearby declarations and improve readability.
  **CN L249:** 该空行用于分隔相邻声明并提升可读性。
- **EN L250:** This TableGen `def` record introduces `Math_AsinhOp`, which later participates in generated MLIR code.
  **CN L250:** 该 TableGen `def` 记录引入了 `Math_AsinhOp`，后续会参与生成的 MLIR 代码。
- **EN L251:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L251:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L252:** This line contributes implementation detail or declarative structure to the file.
  **CN L252:** 这一行为文件补充了实现细节或声明式结构。

### Lines 253-264 / 第 253-264 行

```tablegen
 253:     Syntax:
 254: 
 255:     ```
 256:     operation ::= ssa-id `=` `math.asinh` ssa-use `:` type
 257:     ```
 258: 
 259:     The `asinh` operation computes the hyperbolic arcus sine of a given value.  It takes
 260:     one operand of floating point type (i.e., scalar, tensor or vector) and returns
 261:     one result of the same type. It has no standard attributes.
 262: 
 263:     Example:
 264: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L253:** This line contributes implementation detail or declarative structure to the file.
  **CN L253:** 这一行为文件补充了实现细节或声明式结构。
- **EN L254:** Blank line used to separate nearby declarations and improve readability.
  **CN L254:** 该空行用于分隔相邻声明并提升可读性。
- **EN L255:** This line contributes implementation detail or declarative structure to the file.
  **CN L255:** 这一行为文件补充了实现细节或声明式结构。
- **EN L256:** This line contributes implementation detail or declarative structure to the file.
  **CN L256:** 这一行为文件补充了实现细节或声明式结构。
- **EN L257:** This line contributes implementation detail or declarative structure to the file.
  **CN L257:** 这一行为文件补充了实现细节或声明式结构。
- **EN L258:** Blank line used to separate nearby declarations and improve readability.
  **CN L258:** 该空行用于分隔相邻声明并提升可读性。
- **EN L259:** This line contributes implementation detail or declarative structure to the file.
  **CN L259:** 这一行为文件补充了实现细节或声明式结构。
- **EN L260:** This line contributes to the declaration or call of `type`.
  **CN L260:** 这一行为 `type` 的声明或调用提供内容。
- **EN L261:** This line contributes implementation detail or declarative structure to the file.
  **CN L261:** 这一行为文件补充了实现细节或声明式结构。
- **EN L262:** Blank line used to separate nearby declarations and improve readability.
  **CN L262:** 该空行用于分隔相邻声明并提升可读性。
- **EN L263:** This line contributes implementation detail or declarative structure to the file.
  **CN L263:** 这一行为文件补充了实现细节或声明式结构。
- **EN L264:** Blank line used to separate nearby declarations and improve readability.
  **CN L264:** 该空行用于分隔相邻声明并提升可读性。

### Lines 265-276 / 第 265-276 行

```tablegen
 265:     ```mlir
 266:     // Hyperbolic arcus sine of scalar value.
 267:     %a = math.asinh %b : f64
 268:     ```
 269:   }];
 270:   let hasFolder = 1;
 271: }
 272: 
 273: //===----------------------------------------------------------------------===//
 274: // AtanOp
 275: //===----------------------------------------------------------------------===//
 276: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L265:** This line contributes implementation detail or declarative structure to the file.
  **CN L265:** 这一行为文件补充了实现细节或声明式结构。
- **EN L266:** This comment states: “Hyperbolic arcus sine of scalar value.”, documenting the intent of the surrounding code.
  **CN L266:** 该注释写道：“Hyperbolic arcus sine of scalar value.”，用于说明周围代码的意图。
- **EN L267:** This line contributes implementation detail or declarative structure to the file.
  **CN L267:** 这一行为文件补充了实现细节或声明式结构。
- **EN L268:** This line contributes implementation detail or declarative structure to the file.
  **CN L268:** 这一行为文件补充了实现细节或声明式结构。
- **EN L269:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L269:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L270:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L270:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L271:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L271:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L272:** Blank line used to separate nearby declarations and improve readability.
  **CN L272:** 该空行用于分隔相邻声明并提升可读性。
- **EN L273:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L273:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L274:** This comment states: “AtanOp”, documenting the intent of the surrounding code.
  **CN L274:** 该注释写道：“AtanOp”，用于说明周围代码的意图。
- **EN L275:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L275:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L276:** Blank line used to separate nearby declarations and improve readability.
  **CN L276:** 该空行用于分隔相邻声明并提升可读性。

### Lines 277-288 / 第 277-288 行

```tablegen
 277: def Math_AtanOp : Math_FloatUnaryOp<"atan">{
 278:   let summary = "arcus tangent of the given value";
 279:   let description = [{
 280:     The `atan` operation computes the arcus tangent of a given value.  It takes
 281:     one operand of floating point type (i.e., scalar, tensor or vector) and returns
 282:     one result of the same type. It has no standard attributes.
 283: 
 284:     Example:
 285: 
 286:     ```mlir
 287:     // Arcus tangent of scalar value.
 288:     %a = math.atan %b : f64
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L277:** This TableGen `def` record introduces `Math_AtanOp`, which later participates in generated MLIR code.
  **CN L277:** 该 TableGen `def` 记录引入了 `Math_AtanOp`，后续会参与生成的 MLIR 代码。
- **EN L278:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L278:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L279:** This line contributes implementation detail or declarative structure to the file.
  **CN L279:** 这一行为文件补充了实现细节或声明式结构。
- **EN L280:** This line contributes implementation detail or declarative structure to the file.
  **CN L280:** 这一行为文件补充了实现细节或声明式结构。
- **EN L281:** This line contributes to the declaration or call of `type`.
  **CN L281:** 这一行为 `type` 的声明或调用提供内容。
- **EN L282:** This line contributes implementation detail or declarative structure to the file.
  **CN L282:** 这一行为文件补充了实现细节或声明式结构。
- **EN L283:** Blank line used to separate nearby declarations and improve readability.
  **CN L283:** 该空行用于分隔相邻声明并提升可读性。
- **EN L284:** This line contributes implementation detail or declarative structure to the file.
  **CN L284:** 这一行为文件补充了实现细节或声明式结构。
- **EN L285:** Blank line used to separate nearby declarations and improve readability.
  **CN L285:** 该空行用于分隔相邻声明并提升可读性。
- **EN L286:** This line contributes implementation detail or declarative structure to the file.
  **CN L286:** 这一行为文件补充了实现细节或声明式结构。
- **EN L287:** This comment states: “Arcus tangent of scalar value.”, documenting the intent of the surrounding code.
  **CN L287:** 该注释写道：“Arcus tangent of scalar value.”，用于说明周围代码的意图。
- **EN L288:** This line contributes implementation detail or declarative structure to the file.
  **CN L288:** 这一行为文件补充了实现细节或声明式结构。

### Lines 289-300 / 第 289-300 行

```tablegen
 289:     ```
 290:   }];
 291:   let hasFolder = 1;
 292: }
 293: 
 294: //===----------------------------------------------------------------------===//
 295: // AtanhOp
 296: //===----------------------------------------------------------------------===//
 297: 
 298: def Math_AtanhOp : Math_FloatUnaryOp<"atanh">{
 299:   let summary = "hyperbolic arcus tangent of the given value";
 300:   let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L289:** This line contributes implementation detail or declarative structure to the file.
  **CN L289:** 这一行为文件补充了实现细节或声明式结构。
- **EN L290:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L290:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L291:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L291:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L292:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L292:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L293:** Blank line used to separate nearby declarations and improve readability.
  **CN L293:** 该空行用于分隔相邻声明并提升可读性。
- **EN L294:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L294:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L295:** This comment states: “AtanhOp”, documenting the intent of the surrounding code.
  **CN L295:** 该注释写道：“AtanhOp”，用于说明周围代码的意图。
- **EN L296:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L296:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L297:** Blank line used to separate nearby declarations and improve readability.
  **CN L297:** 该空行用于分隔相邻声明并提升可读性。
- **EN L298:** This TableGen `def` record introduces `Math_AtanhOp`, which later participates in generated MLIR code.
  **CN L298:** 该 TableGen `def` 记录引入了 `Math_AtanhOp`，后续会参与生成的 MLIR 代码。
- **EN L299:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L299:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L300:** This line contributes implementation detail or declarative structure to the file.
  **CN L300:** 这一行为文件补充了实现细节或声明式结构。

### Lines 301-312 / 第 301-312 行

```tablegen
 301:     Syntax:
 302: 
 303:     ```
 304:     operation ::= ssa-id `=` `math.atanh` ssa-use `:` type
 305:     ```
 306: 
 307:     The `atanh` operation computes the hyperbolic arcus tangent of a given value.  It takes
 308:     one operand of floating point type (i.e., scalar, tensor or vector) and returns
 309:     one result of the same type. It has no standard attributes.
 310: 
 311:     Example:
 312: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L301:** This line contributes implementation detail or declarative structure to the file.
  **CN L301:** 这一行为文件补充了实现细节或声明式结构。
- **EN L302:** Blank line used to separate nearby declarations and improve readability.
  **CN L302:** 该空行用于分隔相邻声明并提升可读性。
- **EN L303:** This line contributes implementation detail or declarative structure to the file.
  **CN L303:** 这一行为文件补充了实现细节或声明式结构。
- **EN L304:** This line contributes implementation detail or declarative structure to the file.
  **CN L304:** 这一行为文件补充了实现细节或声明式结构。
- **EN L305:** This line contributes implementation detail or declarative structure to the file.
  **CN L305:** 这一行为文件补充了实现细节或声明式结构。
- **EN L306:** Blank line used to separate nearby declarations and improve readability.
  **CN L306:** 该空行用于分隔相邻声明并提升可读性。
- **EN L307:** This line contributes implementation detail or declarative structure to the file.
  **CN L307:** 这一行为文件补充了实现细节或声明式结构。
- **EN L308:** This line contributes to the declaration or call of `type`.
  **CN L308:** 这一行为 `type` 的声明或调用提供内容。
- **EN L309:** This line contributes implementation detail or declarative structure to the file.
  **CN L309:** 这一行为文件补充了实现细节或声明式结构。
- **EN L310:** Blank line used to separate nearby declarations and improve readability.
  **CN L310:** 该空行用于分隔相邻声明并提升可读性。
- **EN L311:** This line contributes implementation detail or declarative structure to the file.
  **CN L311:** 这一行为文件补充了实现细节或声明式结构。
- **EN L312:** Blank line used to separate nearby declarations and improve readability.
  **CN L312:** 该空行用于分隔相邻声明并提升可读性。

### Lines 313-324 / 第 313-324 行

```tablegen
 313:     ```mlir
 314:     // Hyperbolic arcus tangent of scalar value.
 315:     %a = math.atanh %b : f64
 316:     ```
 317:   }];
 318:   let hasFolder = 1;
 319: }
 320: 
 321: //===----------------------------------------------------------------------===//
 322: // Atan2Op
 323: //===----------------------------------------------------------------------===//
 324: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L313:** This line contributes implementation detail or declarative structure to the file.
  **CN L313:** 这一行为文件补充了实现细节或声明式结构。
- **EN L314:** This comment states: “Hyperbolic arcus tangent of scalar value.”, documenting the intent of the surrounding code.
  **CN L314:** 该注释写道：“Hyperbolic arcus tangent of scalar value.”，用于说明周围代码的意图。
- **EN L315:** This line contributes implementation detail or declarative structure to the file.
  **CN L315:** 这一行为文件补充了实现细节或声明式结构。
- **EN L316:** This line contributes implementation detail or declarative structure to the file.
  **CN L316:** 这一行为文件补充了实现细节或声明式结构。
- **EN L317:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L317:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L318:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L318:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L319:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L319:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L320:** Blank line used to separate nearby declarations and improve readability.
  **CN L320:** 该空行用于分隔相邻声明并提升可读性。
- **EN L321:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L321:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L322:** This comment states: “Atan2Op”, documenting the intent of the surrounding code.
  **CN L322:** 该注释写道：“Atan2Op”，用于说明周围代码的意图。
- **EN L323:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L323:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L324:** Blank line used to separate nearby declarations and improve readability.
  **CN L324:** 该空行用于分隔相邻声明并提升可读性。

### Lines 325-336 / 第 325-336 行

```tablegen
 325: def Math_Atan2Op : Math_FloatBinaryOp<"atan2">{
 326:   let summary = "2-argument arcus tangent of the given values";
 327:   let description = [{
 328:     The `atan2` operation takes two operands and returns one result, all of
 329:     which must be of the same type.  The operands must be of floating point type
 330:     (i.e., scalar, tensor or vector).
 331: 
 332:     The 2-argument arcus tangent `atan2(y, x)` returns the angle in the
 333:     Euclidian plane between the positive x-axis and the ray through the point
 334:     (x, y).  It is a generalization of the 1-argument arcus tangent which
 335:     returns the angle on the basis of the ratio y/x.
 336: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L325:** This TableGen `def` record introduces `Math_Atan2Op`, which later participates in generated MLIR code.
  **CN L325:** 该 TableGen `def` 记录引入了 `Math_Atan2Op`，后续会参与生成的 MLIR 代码。
- **EN L326:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L326:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L327:** This line contributes implementation detail or declarative structure to the file.
  **CN L327:** 这一行为文件补充了实现细节或声明式结构。
- **EN L328:** This line contributes implementation detail or declarative structure to the file.
  **CN L328:** 这一行为文件补充了实现细节或声明式结构。
- **EN L329:** This line contributes implementation detail or declarative structure to the file.
  **CN L329:** 这一行为文件补充了实现细节或声明式结构。
- **EN L330:** This line contributes implementation detail or declarative structure to the file.
  **CN L330:** 这一行为文件补充了实现细节或声明式结构。
- **EN L331:** Blank line used to separate nearby declarations and improve readability.
  **CN L331:** 该空行用于分隔相邻声明并提升可读性。
- **EN L332:** This line contributes to the declaration or call of `atan2`.
  **CN L332:** 这一行为 `atan2` 的声明或调用提供内容。
- **EN L333:** This line contributes implementation detail or declarative structure to the file.
  **CN L333:** 这一行为文件补充了实现细节或声明式结构。
- **EN L334:** This line contributes implementation detail or declarative structure to the file.
  **CN L334:** 这一行为文件补充了实现细节或声明式结构。
- **EN L335:** This line contributes implementation detail or declarative structure to the file.
  **CN L335:** 这一行为文件补充了实现细节或声明式结构。
- **EN L336:** Blank line used to separate nearby declarations and improve readability.
  **CN L336:** 该空行用于分隔相邻声明并提升可读性。

### Lines 337-348 / 第 337-348 行

```tablegen
 337:     See also https://en.wikipedia.org/wiki/Atan2
 338: 
 339:     Example:
 340: 
 341:     ```mlir
 342:     // Scalar variant.
 343:     %a = math.atan2 %b, %c : f32
 344:     ```
 345:   }];
 346:   let hasFolder = 1;
 347: }
 348: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L337:** This line contributes implementation detail or declarative structure to the file.
  **CN L337:** 这一行为文件补充了实现细节或声明式结构。
- **EN L338:** Blank line used to separate nearby declarations and improve readability.
  **CN L338:** 该空行用于分隔相邻声明并提升可读性。
- **EN L339:** This line contributes implementation detail or declarative structure to the file.
  **CN L339:** 这一行为文件补充了实现细节或声明式结构。
- **EN L340:** Blank line used to separate nearby declarations and improve readability.
  **CN L340:** 该空行用于分隔相邻声明并提升可读性。
- **EN L341:** This line contributes implementation detail or declarative structure to the file.
  **CN L341:** 这一行为文件补充了实现细节或声明式结构。
- **EN L342:** This comment states: “Scalar variant.”, documenting the intent of the surrounding code.
  **CN L342:** 该注释写道：“Scalar variant.”，用于说明周围代码的意图。
- **EN L343:** This line contributes implementation detail or declarative structure to the file.
  **CN L343:** 这一行为文件补充了实现细节或声明式结构。
- **EN L344:** This line contributes implementation detail or declarative structure to the file.
  **CN L344:** 这一行为文件补充了实现细节或声明式结构。
- **EN L345:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L345:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L346:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L346:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L347:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L347:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L348:** Blank line used to separate nearby declarations and improve readability.
  **CN L348:** 该空行用于分隔相邻声明并提升可读性。

### Lines 349-360 / 第 349-360 行

```tablegen
 349: //===----------------------------------------------------------------------===//
 350: // CbrtOp
 351: //===----------------------------------------------------------------------===//
 352: 
 353: def Math_CbrtOp : Math_FloatUnaryOp<"cbrt"> {
 354:   let summary = "cube root of the specified value";
 355:   let description = [{
 356:     The `cbrt` operation computes the cube root. It takes one operand of
 357:     floating point type (i.e., scalar, tensor or vector) and returns one result
 358:     of the same type. It has no standard attributes.
 359: 
 360:     Example:
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L349:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L349:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L350:** This comment states: “CbrtOp”, documenting the intent of the surrounding code.
  **CN L350:** 该注释写道：“CbrtOp”，用于说明周围代码的意图。
- **EN L351:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L351:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L352:** Blank line used to separate nearby declarations and improve readability.
  **CN L352:** 该空行用于分隔相邻声明并提升可读性。
- **EN L353:** This TableGen `def` record introduces `Math_CbrtOp`, which later participates in generated MLIR code.
  **CN L353:** 该 TableGen `def` 记录引入了 `Math_CbrtOp`，后续会参与生成的 MLIR 代码。
- **EN L354:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L354:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L355:** This line contributes implementation detail or declarative structure to the file.
  **CN L355:** 这一行为文件补充了实现细节或声明式结构。
- **EN L356:** This line contributes implementation detail or declarative structure to the file.
  **CN L356:** 这一行为文件补充了实现细节或声明式结构。
- **EN L357:** This line contributes to the declaration or call of `type`.
  **CN L357:** 这一行为 `type` 的声明或调用提供内容。
- **EN L358:** This line contributes implementation detail or declarative structure to the file.
  **CN L358:** 这一行为文件补充了实现细节或声明式结构。
- **EN L359:** Blank line used to separate nearby declarations and improve readability.
  **CN L359:** 该空行用于分隔相邻声明并提升可读性。
- **EN L360:** This line contributes implementation detail or declarative structure to the file.
  **CN L360:** 这一行为文件补充了实现细节或声明式结构。

### Lines 361-372 / 第 361-372 行

```tablegen
 361: 
 362:     ```mlir
 363:     // Scalar cube root value.
 364:     %a = math.cbrt %b : f64
 365:     ```
 366: 
 367:     Note: This op is not equivalent to powf(..., 1/3.0).
 368:   }];
 369: 
 370:   let hasFolder = 1;
 371: }
 372: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L361:** Blank line used to separate nearby declarations and improve readability.
  **CN L361:** 该空行用于分隔相邻声明并提升可读性。
- **EN L362:** This line contributes implementation detail or declarative structure to the file.
  **CN L362:** 这一行为文件补充了实现细节或声明式结构。
- **EN L363:** This comment states: “Scalar cube root value.”, documenting the intent of the surrounding code.
  **CN L363:** 该注释写道：“Scalar cube root value.”，用于说明周围代码的意图。
- **EN L364:** This line contributes implementation detail or declarative structure to the file.
  **CN L364:** 这一行为文件补充了实现细节或声明式结构。
- **EN L365:** This line contributes implementation detail or declarative structure to the file.
  **CN L365:** 这一行为文件补充了实现细节或声明式结构。
- **EN L366:** Blank line used to separate nearby declarations and improve readability.
  **CN L366:** 该空行用于分隔相邻声明并提升可读性。
- **EN L367:** This line contributes to the declaration or call of `powf`.
  **CN L367:** 这一行为 `powf` 的声明或调用提供内容。
- **EN L368:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L368:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L369:** Blank line used to separate nearby declarations and improve readability.
  **CN L369:** 该空行用于分隔相邻声明并提升可读性。
- **EN L370:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L370:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L371:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L371:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L372:** Blank line used to separate nearby declarations and improve readability.
  **CN L372:** 该空行用于分隔相邻声明并提升可读性。

### Lines 373-384 / 第 373-384 行

```tablegen
 373: //===----------------------------------------------------------------------===//
 374: // CeilOp
 375: //===----------------------------------------------------------------------===//
 376: 
 377: def Math_CeilOp : Math_FloatUnaryOp<"ceil"> {
 378:   let summary = "ceiling of the specified value";
 379:   let description = [{
 380:     The `ceil` operation computes the ceiling of a given value. It takes one
 381:     operand of floating point type (i.e., scalar, tensor or vector) and returns one
 382:     result of the same type.  It has no standard attributes.
 383: 
 384:     Example:
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L373:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L373:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L374:** This comment states: “CeilOp”, documenting the intent of the surrounding code.
  **CN L374:** 该注释写道：“CeilOp”，用于说明周围代码的意图。
- **EN L375:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L375:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L376:** Blank line used to separate nearby declarations and improve readability.
  **CN L376:** 该空行用于分隔相邻声明并提升可读性。
- **EN L377:** This TableGen `def` record introduces `Math_CeilOp`, which later participates in generated MLIR code.
  **CN L377:** 该 TableGen `def` 记录引入了 `Math_CeilOp`，后续会参与生成的 MLIR 代码。
- **EN L378:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L378:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L379:** This line contributes implementation detail or declarative structure to the file.
  **CN L379:** 这一行为文件补充了实现细节或声明式结构。
- **EN L380:** This line contributes implementation detail or declarative structure to the file.
  **CN L380:** 这一行为文件补充了实现细节或声明式结构。
- **EN L381:** This line contributes to the declaration or call of `type`.
  **CN L381:** 这一行为 `type` 的声明或调用提供内容。
- **EN L382:** This line contributes implementation detail or declarative structure to the file.
  **CN L382:** 这一行为文件补充了实现细节或声明式结构。
- **EN L383:** Blank line used to separate nearby declarations and improve readability.
  **CN L383:** 该空行用于分隔相邻声明并提升可读性。
- **EN L384:** This line contributes implementation detail or declarative structure to the file.
  **CN L384:** 这一行为文件补充了实现细节或声明式结构。

### Lines 385-396 / 第 385-396 行

```tablegen
 385: 
 386:     ```mlir
 387:     // Scalar ceiling value.
 388:     %a = math.ceil %b : f64
 389:     ```
 390:   }];
 391:   let hasFolder = 1;
 392: }
 393: 
 394: //===----------------------------------------------------------------------===//
 395: // ClampFOp
 396: //===----------------------------------------------------------------------===//
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L385:** Blank line used to separate nearby declarations and improve readability.
  **CN L385:** 该空行用于分隔相邻声明并提升可读性。
- **EN L386:** This line contributes implementation detail or declarative structure to the file.
  **CN L386:** 这一行为文件补充了实现细节或声明式结构。
- **EN L387:** This comment states: “Scalar ceiling value.”, documenting the intent of the surrounding code.
  **CN L387:** 该注释写道：“Scalar ceiling value.”，用于说明周围代码的意图。
- **EN L388:** This line contributes implementation detail or declarative structure to the file.
  **CN L388:** 这一行为文件补充了实现细节或声明式结构。
- **EN L389:** This line contributes implementation detail or declarative structure to the file.
  **CN L389:** 这一行为文件补充了实现细节或声明式结构。
- **EN L390:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L390:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L391:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L391:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L392:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L392:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L393:** Blank line used to separate nearby declarations and improve readability.
  **CN L393:** 该空行用于分隔相邻声明并提升可读性。
- **EN L394:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L394:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L395:** This comment states: “ClampFOp”, documenting the intent of the surrounding code.
  **CN L395:** 该注释写道：“ClampFOp”，用于说明周围代码的意图。
- **EN L396:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L396:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 397-408 / 第 397-408 行

```tablegen
 397: 
 398: def Math_ClampFOp : Math_FloatTernaryOp<"clampf"> {
 399:   let summary = "floating point clamping operation";
 400:   let description = [{
 401:     The `clampf` operation takes three operands and returns one result, each of
 402:     these is required to be the same type. Operands must be of floating point type
 403:     (i.e., scalar, tensor or vector).
 404: 
 405:     The semantics of the operation are described by:
 406:     ```
 407:       clampf(value, min, max) = maxf(minf(value, max), min)
 408:     ```
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L397:** Blank line used to separate nearby declarations and improve readability.
  **CN L397:** 该空行用于分隔相邻声明并提升可读性。
- **EN L398:** This TableGen `def` record introduces `Math_ClampFOp`, which later participates in generated MLIR code.
  **CN L398:** 该 TableGen `def` 记录引入了 `Math_ClampFOp`，后续会参与生成的 MLIR 代码。
- **EN L399:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L399:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L400:** This line contributes implementation detail or declarative structure to the file.
  **CN L400:** 这一行为文件补充了实现细节或声明式结构。
- **EN L401:** This line contributes implementation detail or declarative structure to the file.
  **CN L401:** 这一行为文件补充了实现细节或声明式结构。
- **EN L402:** This line contributes implementation detail or declarative structure to the file.
  **CN L402:** 这一行为文件补充了实现细节或声明式结构。
- **EN L403:** This line contributes implementation detail or declarative structure to the file.
  **CN L403:** 这一行为文件补充了实现细节或声明式结构。
- **EN L404:** Blank line used to separate nearby declarations and improve readability.
  **CN L404:** 该空行用于分隔相邻声明并提升可读性。
- **EN L405:** This line contributes implementation detail or declarative structure to the file.
  **CN L405:** 这一行为文件补充了实现细节或声明式结构。
- **EN L406:** This line contributes implementation detail or declarative structure to the file.
  **CN L406:** 这一行为文件补充了实现细节或声明式结构。
- **EN L407:** This line contributes to the declaration or call of `clampf`.
  **CN L407:** 这一行为 `clampf` 的声明或调用提供内容。
- **EN L408:** This line contributes implementation detail or declarative structure to the file.
  **CN L408:** 这一行为文件补充了实现细节或声明式结构。

### Lines 409-420 / 第 409-420 行

```tablegen
 409:     If `min > max` the resulting value is poison.
 410: 
 411:     Example:
 412: 
 413:     ```mlir
 414:     %d = math.clampf %value to [%min, %max] : f64
 415:     ```
 416:   }];
 417:   let arguments = (ins FloatLike:$value, FloatLike:$min, FloatLike:$max,
 418:       DefaultValuedAttr<Arith_FastMathAttr,
 419:                         "::mlir::arith::FastMathFlags::none">:$fastmath);
 420:   let assemblyFormat = [{
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L409:** This line contributes implementation detail or declarative structure to the file.
  **CN L409:** 这一行为文件补充了实现细节或声明式结构。
- **EN L410:** Blank line used to separate nearby declarations and improve readability.
  **CN L410:** 该空行用于分隔相邻声明并提升可读性。
- **EN L411:** This line contributes implementation detail or declarative structure to the file.
  **CN L411:** 这一行为文件补充了实现细节或声明式结构。
- **EN L412:** Blank line used to separate nearby declarations and improve readability.
  **CN L412:** 该空行用于分隔相邻声明并提升可读性。
- **EN L413:** This line contributes implementation detail or declarative structure to the file.
  **CN L413:** 这一行为文件补充了实现细节或声明式结构。
- **EN L414:** This line contributes implementation detail or declarative structure to the file.
  **CN L414:** 这一行为文件补充了实现细节或声明式结构。
- **EN L415:** This line contributes implementation detail or declarative structure to the file.
  **CN L415:** 这一行为文件补充了实现细节或声明式结构。
- **EN L416:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L416:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L417:** This line contributes implementation detail or declarative structure to the file.
  **CN L417:** 这一行为文件补充了实现细节或声明式结构。
- **EN L418:** This line contributes implementation detail or declarative structure to the file.
  **CN L418:** 这一行为文件补充了实现细节或声明式结构。
- **EN L419:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L419:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L420:** This line contributes implementation detail or declarative structure to the file.
  **CN L420:** 这一行为文件补充了实现细节或声明式结构。

### Lines 421-432 / 第 421-432 行

```tablegen
 421:     $value `to` ` ` `[` $min `,` $max `]` (`fastmath` `` $fastmath^)?
 422:     attr-dict `:` type($result)
 423:   }];
 424: }
 425: 
 426: //===----------------------------------------------------------------------===//
 427: // CopySignOp
 428: //===----------------------------------------------------------------------===//
 429: 
 430: def Math_CopySignOp : Math_FloatBinaryOp<"copysign"> {
 431:   let summary = "A copysign operation";
 432:   let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L421:** This line contributes implementation detail or declarative structure to the file.
  **CN L421:** 这一行为文件补充了实现细节或声明式结构。
- **EN L422:** This line contributes to the declaration or call of `type`.
  **CN L422:** 这一行为 `type` 的声明或调用提供内容。
- **EN L423:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L423:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L424:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L424:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L425:** Blank line used to separate nearby declarations and improve readability.
  **CN L425:** 该空行用于分隔相邻声明并提升可读性。
- **EN L426:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L426:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L427:** This comment states: “CopySignOp”, documenting the intent of the surrounding code.
  **CN L427:** 该注释写道：“CopySignOp”，用于说明周围代码的意图。
- **EN L428:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L428:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L429:** Blank line used to separate nearby declarations and improve readability.
  **CN L429:** 该空行用于分隔相邻声明并提升可读性。
- **EN L430:** This TableGen `def` record introduces `Math_CopySignOp`, which later participates in generated MLIR code.
  **CN L430:** 该 TableGen `def` 记录引入了 `Math_CopySignOp`，后续会参与生成的 MLIR 代码。
- **EN L431:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L431:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L432:** This line contributes implementation detail or declarative structure to the file.
  **CN L432:** 这一行为文件补充了实现细节或声明式结构。

### Lines 433-444 / 第 433-444 行

```tablegen
 433:     The `copysign` returns a value with the magnitude of the first operand and
 434:     the sign of the second operand. It takes two operands and returns one result of
 435:     the same type. The operands must be of floating point type (i.e., scalar,
 436:     tensor or vector). It has no standard attributes.
 437: 
 438:     Example:
 439: 
 440:     ```mlir
 441:     // Scalar copysign value.
 442:     %a = math.copysign %b, %c : f64
 443:     ```
 444:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L433:** This line contributes implementation detail or declarative structure to the file.
  **CN L433:** 这一行为文件补充了实现细节或声明式结构。
- **EN L434:** This line contributes implementation detail or declarative structure to the file.
  **CN L434:** 这一行为文件补充了实现细节或声明式结构。
- **EN L435:** This line contributes to the declaration or call of `type`.
  **CN L435:** 这一行为 `type` 的声明或调用提供内容。
- **EN L436:** This line contributes implementation detail or declarative structure to the file.
  **CN L436:** 这一行为文件补充了实现细节或声明式结构。
- **EN L437:** Blank line used to separate nearby declarations and improve readability.
  **CN L437:** 该空行用于分隔相邻声明并提升可读性。
- **EN L438:** This line contributes implementation detail or declarative structure to the file.
  **CN L438:** 这一行为文件补充了实现细节或声明式结构。
- **EN L439:** Blank line used to separate nearby declarations and improve readability.
  **CN L439:** 该空行用于分隔相邻声明并提升可读性。
- **EN L440:** This line contributes implementation detail or declarative structure to the file.
  **CN L440:** 这一行为文件补充了实现细节或声明式结构。
- **EN L441:** This comment states: “Scalar copysign value.”, documenting the intent of the surrounding code.
  **CN L441:** 该注释写道：“Scalar copysign value.”，用于说明周围代码的意图。
- **EN L442:** This line contributes implementation detail or declarative structure to the file.
  **CN L442:** 这一行为文件补充了实现细节或声明式结构。
- **EN L443:** This line contributes implementation detail or declarative structure to the file.
  **CN L443:** 这一行为文件补充了实现细节或声明式结构。
- **EN L444:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L444:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 445-456 / 第 445-456 行

```tablegen
 445:   let hasFolder = 1;
 446: }
 447: 
 448: //===----------------------------------------------------------------------===//
 449: // CosOp
 450: //===----------------------------------------------------------------------===//
 451: 
 452: def Math_CosOp : Math_FloatUnaryOp<"cos"> {
 453:   let summary = "cosine of the specified value";
 454:   let description = [{
 455:     The `cos` operation computes the cosine of a given value. It takes one
 456:     operand of floating point type (i.e., scalar, tensor or vector) and returns one
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L445:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L445:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L446:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L446:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L447:** Blank line used to separate nearby declarations and improve readability.
  **CN L447:** 该空行用于分隔相邻声明并提升可读性。
- **EN L448:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L448:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L449:** This comment states: “CosOp”, documenting the intent of the surrounding code.
  **CN L449:** 该注释写道：“CosOp”，用于说明周围代码的意图。
- **EN L450:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L450:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L451:** Blank line used to separate nearby declarations and improve readability.
  **CN L451:** 该空行用于分隔相邻声明并提升可读性。
- **EN L452:** This TableGen `def` record introduces `Math_CosOp`, which later participates in generated MLIR code.
  **CN L452:** 该 TableGen `def` 记录引入了 `Math_CosOp`，后续会参与生成的 MLIR 代码。
- **EN L453:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L453:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L454:** This line contributes implementation detail or declarative structure to the file.
  **CN L454:** 这一行为文件补充了实现细节或声明式结构。
- **EN L455:** This line contributes implementation detail or declarative structure to the file.
  **CN L455:** 这一行为文件补充了实现细节或声明式结构。
- **EN L456:** This line contributes to the declaration or call of `type`.
  **CN L456:** 这一行为 `type` 的声明或调用提供内容。

### Lines 457-468 / 第 457-468 行

```tablegen
 457:     result of the same type.  It has no standard attributes.
 458: 
 459:     Example:
 460: 
 461:     ```mlir
 462:     // Scalar cosine value.
 463:     %a = math.cos %b : f64
 464:     ```
 465:   }];
 466:   let hasFolder = 1;
 467: }
 468: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L457:** This line contributes implementation detail or declarative structure to the file.
  **CN L457:** 这一行为文件补充了实现细节或声明式结构。
- **EN L458:** Blank line used to separate nearby declarations and improve readability.
  **CN L458:** 该空行用于分隔相邻声明并提升可读性。
- **EN L459:** This line contributes implementation detail or declarative structure to the file.
  **CN L459:** 这一行为文件补充了实现细节或声明式结构。
- **EN L460:** Blank line used to separate nearby declarations and improve readability.
  **CN L460:** 该空行用于分隔相邻声明并提升可读性。
- **EN L461:** This line contributes implementation detail or declarative structure to the file.
  **CN L461:** 这一行为文件补充了实现细节或声明式结构。
- **EN L462:** This comment states: “Scalar cosine value.”, documenting the intent of the surrounding code.
  **CN L462:** 该注释写道：“Scalar cosine value.”，用于说明周围代码的意图。
- **EN L463:** This line contributes implementation detail or declarative structure to the file.
  **CN L463:** 这一行为文件补充了实现细节或声明式结构。
- **EN L464:** This line contributes implementation detail or declarative structure to the file.
  **CN L464:** 这一行为文件补充了实现细节或声明式结构。
- **EN L465:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L465:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L466:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L466:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L467:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L467:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L468:** Blank line used to separate nearby declarations and improve readability.
  **CN L468:** 该空行用于分隔相邻声明并提升可读性。

### Lines 469-480 / 第 469-480 行

```tablegen
 469: //===----------------------------------------------------------------------===//
 470: // AcosOp
 471: //===----------------------------------------------------------------------===//
 472: 
 473: def Math_AcosOp : Math_FloatUnaryOp<"acos"> {
 474:   let summary = "arcus cosine of the specified value";
 475:   let description = [{
 476:     The `acos` operation computes the arcus cosine of a given value. It takes one
 477:     operand of floating point type (i.e., scalar, tensor or vector) and returns one
 478:     result of the same type.  It has no standard attributes.
 479: 
 480:     Example:
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L469:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L469:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L470:** This comment states: “AcosOp”, documenting the intent of the surrounding code.
  **CN L470:** 该注释写道：“AcosOp”，用于说明周围代码的意图。
- **EN L471:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L471:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L472:** Blank line used to separate nearby declarations and improve readability.
  **CN L472:** 该空行用于分隔相邻声明并提升可读性。
- **EN L473:** This TableGen `def` record introduces `Math_AcosOp`, which later participates in generated MLIR code.
  **CN L473:** 该 TableGen `def` 记录引入了 `Math_AcosOp`，后续会参与生成的 MLIR 代码。
- **EN L474:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L474:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L475:** This line contributes implementation detail or declarative structure to the file.
  **CN L475:** 这一行为文件补充了实现细节或声明式结构。
- **EN L476:** This line contributes implementation detail or declarative structure to the file.
  **CN L476:** 这一行为文件补充了实现细节或声明式结构。
- **EN L477:** This line contributes to the declaration or call of `type`.
  **CN L477:** 这一行为 `type` 的声明或调用提供内容。
- **EN L478:** This line contributes implementation detail or declarative structure to the file.
  **CN L478:** 这一行为文件补充了实现细节或声明式结构。
- **EN L479:** Blank line used to separate nearby declarations and improve readability.
  **CN L479:** 该空行用于分隔相邻声明并提升可读性。
- **EN L480:** This line contributes implementation detail or declarative structure to the file.
  **CN L480:** 这一行为文件补充了实现细节或声明式结构。

### Lines 481-492 / 第 481-492 行

```tablegen
 481: 
 482:     ```mlir
 483:     // Scalar arcus cosine value.
 484:     %a = math.acos %b : f64
 485:     ```
 486:   }];
 487:   let hasFolder = 1;
 488: }
 489: 
 490: //===----------------------------------------------------------------------===//
 491: // CoshOp
 492: //===----------------------------------------------------------------------===//
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L481:** Blank line used to separate nearby declarations and improve readability.
  **CN L481:** 该空行用于分隔相邻声明并提升可读性。
- **EN L482:** This line contributes implementation detail or declarative structure to the file.
  **CN L482:** 这一行为文件补充了实现细节或声明式结构。
- **EN L483:** This comment states: “Scalar arcus cosine value.”, documenting the intent of the surrounding code.
  **CN L483:** 该注释写道：“Scalar arcus cosine value.”，用于说明周围代码的意图。
- **EN L484:** This line contributes implementation detail or declarative structure to the file.
  **CN L484:** 这一行为文件补充了实现细节或声明式结构。
- **EN L485:** This line contributes implementation detail or declarative structure to the file.
  **CN L485:** 这一行为文件补充了实现细节或声明式结构。
- **EN L486:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L486:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L487:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L487:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L488:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L488:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L489:** Blank line used to separate nearby declarations and improve readability.
  **CN L489:** 该空行用于分隔相邻声明并提升可读性。
- **EN L490:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L490:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L491:** This comment states: “CoshOp”, documenting the intent of the surrounding code.
  **CN L491:** 该注释写道：“CoshOp”，用于说明周围代码的意图。
- **EN L492:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L492:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 493-504 / 第 493-504 行

```tablegen
 493: 
 494: def Math_CoshOp : Math_FloatUnaryOp<"cosh"> {
 495:   let summary = "hyperbolic cosine of the specified value";
 496:   let description = [{
 497:     The `cosh` operation computes the hyperbolic cosine. It takes one operand
 498:     of floating point type (i.e., scalar, tensor or vector) and returns one
 499:     result of the same type. It has no standard attributes.
 500: 
 501:     Example:
 502: 
 503:     ```mlir
 504:     // Scalar hyperbolic cosine value.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L493:** Blank line used to separate nearby declarations and improve readability.
  **CN L493:** 该空行用于分隔相邻声明并提升可读性。
- **EN L494:** This TableGen `def` record introduces `Math_CoshOp`, which later participates in generated MLIR code.
  **CN L494:** 该 TableGen `def` 记录引入了 `Math_CoshOp`，后续会参与生成的 MLIR 代码。
- **EN L495:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L495:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L496:** This line contributes implementation detail or declarative structure to the file.
  **CN L496:** 这一行为文件补充了实现细节或声明式结构。
- **EN L497:** This line contributes implementation detail or declarative structure to the file.
  **CN L497:** 这一行为文件补充了实现细节或声明式结构。
- **EN L498:** This line contributes to the declaration or call of `type`.
  **CN L498:** 这一行为 `type` 的声明或调用提供内容。
- **EN L499:** This line contributes implementation detail or declarative structure to the file.
  **CN L499:** 这一行为文件补充了实现细节或声明式结构。
- **EN L500:** Blank line used to separate nearby declarations and improve readability.
  **CN L500:** 该空行用于分隔相邻声明并提升可读性。
- **EN L501:** This line contributes implementation detail or declarative structure to the file.
  **CN L501:** 这一行为文件补充了实现细节或声明式结构。
- **EN L502:** Blank line used to separate nearby declarations and improve readability.
  **CN L502:** 该空行用于分隔相邻声明并提升可读性。
- **EN L503:** This line contributes implementation detail or declarative structure to the file.
  **CN L503:** 这一行为文件补充了实现细节或声明式结构。
- **EN L504:** This comment states: “Scalar hyperbolic cosine value.”, documenting the intent of the surrounding code.
  **CN L504:** 该注释写道：“Scalar hyperbolic cosine value.”，用于说明周围代码的意图。

### Lines 505-516 / 第 505-516 行

```tablegen
 505:     %a = math.cosh %b : f64
 506:     ```
 507:   }];
 508:   let hasFolder = 1;
 509: }
 510: 
 511: //===----------------------------------------------------------------------===//
 512: // SinOp
 513: //===----------------------------------------------------------------------===//
 514: 
 515: def Math_SinOp : Math_FloatUnaryOp<"sin"> {
 516:   let summary = "sine of the specified value";
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L505:** This line contributes implementation detail or declarative structure to the file.
  **CN L505:** 这一行为文件补充了实现细节或声明式结构。
- **EN L506:** This line contributes implementation detail or declarative structure to the file.
  **CN L506:** 这一行为文件补充了实现细节或声明式结构。
- **EN L507:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L507:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L508:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L508:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L509:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L509:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L510:** Blank line used to separate nearby declarations and improve readability.
  **CN L510:** 该空行用于分隔相邻声明并提升可读性。
- **EN L511:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L511:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L512:** This comment states: “SinOp”, documenting the intent of the surrounding code.
  **CN L512:** 该注释写道：“SinOp”，用于说明周围代码的意图。
- **EN L513:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L513:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L514:** Blank line used to separate nearby declarations and improve readability.
  **CN L514:** 该空行用于分隔相邻声明并提升可读性。
- **EN L515:** This TableGen `def` record introduces `Math_SinOp`, which later participates in generated MLIR code.
  **CN L515:** 该 TableGen `def` 记录引入了 `Math_SinOp`，后续会参与生成的 MLIR 代码。
- **EN L516:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L516:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 517-528 / 第 517-528 行

```tablegen
 517:   let description = [{
 518:     The `sin` operation computes the sine of a given value. It takes one
 519:     operand of floating point type (i.e., scalar, tensor or vector) and returns one
 520:     result of the same type.  It has no standard attributes.
 521: 
 522:     Example:
 523: 
 524:     ```mlir
 525:     // Scalar sine value.
 526:     %a = math.sin %b : f64
 527:     ```
 528:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L517:** This line contributes implementation detail or declarative structure to the file.
  **CN L517:** 这一行为文件补充了实现细节或声明式结构。
- **EN L518:** This line contributes implementation detail or declarative structure to the file.
  **CN L518:** 这一行为文件补充了实现细节或声明式结构。
- **EN L519:** This line contributes to the declaration or call of `type`.
  **CN L519:** 这一行为 `type` 的声明或调用提供内容。
- **EN L520:** This line contributes implementation detail or declarative structure to the file.
  **CN L520:** 这一行为文件补充了实现细节或声明式结构。
- **EN L521:** Blank line used to separate nearby declarations and improve readability.
  **CN L521:** 该空行用于分隔相邻声明并提升可读性。
- **EN L522:** This line contributes implementation detail or declarative structure to the file.
  **CN L522:** 这一行为文件补充了实现细节或声明式结构。
- **EN L523:** Blank line used to separate nearby declarations and improve readability.
  **CN L523:** 该空行用于分隔相邻声明并提升可读性。
- **EN L524:** This line contributes implementation detail or declarative structure to the file.
  **CN L524:** 这一行为文件补充了实现细节或声明式结构。
- **EN L525:** This comment states: “Scalar sine value.”, documenting the intent of the surrounding code.
  **CN L525:** 该注释写道：“Scalar sine value.”，用于说明周围代码的意图。
- **EN L526:** This line contributes implementation detail or declarative structure to the file.
  **CN L526:** 这一行为文件补充了实现细节或声明式结构。
- **EN L527:** This line contributes implementation detail or declarative structure to the file.
  **CN L527:** 这一行为文件补充了实现细节或声明式结构。
- **EN L528:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L528:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 529-540 / 第 529-540 行

```tablegen
 529:   let hasFolder = 1;
 530: }
 531: 
 532: //===----------------------------------------------------------------------===//
 533: // SinhOp
 534: //===----------------------------------------------------------------------===//
 535: 
 536: def Math_SinhOp : Math_FloatUnaryOp<"sinh"> {
 537:   let summary = "hyperbolic sine of the specified value";
 538:   let description = [{
 539:     The `sinh` operation computes the hyperbolic sine. It takes one operand
 540:     of floating point type (i.e., scalar, tensor or vector) and returns one
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L529:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L529:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L530:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L530:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L531:** Blank line used to separate nearby declarations and improve readability.
  **CN L531:** 该空行用于分隔相邻声明并提升可读性。
- **EN L532:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L532:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L533:** This comment states: “SinhOp”, documenting the intent of the surrounding code.
  **CN L533:** 该注释写道：“SinhOp”，用于说明周围代码的意图。
- **EN L534:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L534:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L535:** Blank line used to separate nearby declarations and improve readability.
  **CN L535:** 该空行用于分隔相邻声明并提升可读性。
- **EN L536:** This TableGen `def` record introduces `Math_SinhOp`, which later participates in generated MLIR code.
  **CN L536:** 该 TableGen `def` 记录引入了 `Math_SinhOp`，后续会参与生成的 MLIR 代码。
- **EN L537:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L537:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L538:** This line contributes implementation detail or declarative structure to the file.
  **CN L538:** 这一行为文件补充了实现细节或声明式结构。
- **EN L539:** This line contributes implementation detail or declarative structure to the file.
  **CN L539:** 这一行为文件补充了实现细节或声明式结构。
- **EN L540:** This line contributes to the declaration or call of `type`.
  **CN L540:** 这一行为 `type` 的声明或调用提供内容。

### Lines 541-552 / 第 541-552 行

```tablegen
 541:     result of the same type. It has no standard attributes.
 542: 
 543:     Example:
 544: 
 545:     ```mlir
 546:     // Scalar hyperbolic sine value.
 547:     %a = math.sinh %b : f64
 548:     ```
 549:   }];
 550:   let hasFolder = 1;
 551: }
 552: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L541:** This line contributes implementation detail or declarative structure to the file.
  **CN L541:** 这一行为文件补充了实现细节或声明式结构。
- **EN L542:** Blank line used to separate nearby declarations and improve readability.
  **CN L542:** 该空行用于分隔相邻声明并提升可读性。
- **EN L543:** This line contributes implementation detail or declarative structure to the file.
  **CN L543:** 这一行为文件补充了实现细节或声明式结构。
- **EN L544:** Blank line used to separate nearby declarations and improve readability.
  **CN L544:** 该空行用于分隔相邻声明并提升可读性。
- **EN L545:** This line contributes implementation detail or declarative structure to the file.
  **CN L545:** 这一行为文件补充了实现细节或声明式结构。
- **EN L546:** This comment states: “Scalar hyperbolic sine value.”, documenting the intent of the surrounding code.
  **CN L546:** 该注释写道：“Scalar hyperbolic sine value.”，用于说明周围代码的意图。
- **EN L547:** This line contributes implementation detail or declarative structure to the file.
  **CN L547:** 这一行为文件补充了实现细节或声明式结构。
- **EN L548:** This line contributes implementation detail or declarative structure to the file.
  **CN L548:** 这一行为文件补充了实现细节或声明式结构。
- **EN L549:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L549:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L550:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L550:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L551:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L551:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L552:** Blank line used to separate nearby declarations and improve readability.
  **CN L552:** 该空行用于分隔相邻声明并提升可读性。

### Lines 553-564 / 第 553-564 行

```tablegen
 553: //===----------------------------------------------------------------------===//
 554: // SinCosOp
 555: //===----------------------------------------------------------------------===//
 556: 
 557: def Math_SincosOp : Math_Op<"sincos",
 558:     [SameOperandsAndResultType,
 559:      DeclareOpInterfaceMethods<ArithFastMathInterface>]> {
 560:   let summary = "sine and cosine of the specified value";
 561:   let description = [{
 562:     The `sincos` operation computes both the sine and cosine of a given value
 563:     simultaneously. It takes one operand of floating point type (i.e., scalar,
 564:     tensor or vector) and returns two results of the same type. This operation
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L553:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L553:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L554:** This comment states: “SinCosOp”, documenting the intent of the surrounding code.
  **CN L554:** 该注释写道：“SinCosOp”，用于说明周围代码的意图。
- **EN L555:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L555:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L556:** Blank line used to separate nearby declarations and improve readability.
  **CN L556:** 该空行用于分隔相邻声明并提升可读性。
- **EN L557:** This TableGen `def` record introduces `Math_SincosOp`, which later participates in generated MLIR code.
  **CN L557:** 该 TableGen `def` 记录引入了 `Math_SincosOp`，后续会参与生成的 MLIR 代码。
- **EN L558:** This line contributes implementation detail or declarative structure to the file.
  **CN L558:** 这一行为文件补充了实现细节或声明式结构。
- **EN L559:** This line contributes implementation detail or declarative structure to the file.
  **CN L559:** 这一行为文件补充了实现细节或声明式结构。
- **EN L560:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L560:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L561:** This line contributes implementation detail or declarative structure to the file.
  **CN L561:** 这一行为文件补充了实现细节或声明式结构。
- **EN L562:** This line contributes implementation detail or declarative structure to the file.
  **CN L562:** 这一行为文件补充了实现细节或声明式结构。
- **EN L563:** This line contributes to the declaration or call of `type`.
  **CN L563:** 这一行为 `type` 的声明或调用提供内容。
- **EN L564:** This line contributes implementation detail or declarative structure to the file.
  **CN L564:** 这一行为文件补充了实现细节或声明式结构。

### Lines 565-576 / 第 565-576 行

```tablegen
 565:     can be more efficient than computing sine and cosine separately when both
 566:     values are needed.
 567: 
 568:     Example:
 569: 
 570:     ```mlir
 571:     // Scalar sine and cosine values.
 572:     %sin, %cos = math.sincos %input : f64
 573:     ```
 574:   }];
 575: 
 576:   let arguments = (ins FloatLike:$operand,
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L565:** This line contributes implementation detail or declarative structure to the file.
  **CN L565:** 这一行为文件补充了实现细节或声明式结构。
- **EN L566:** This line contributes implementation detail or declarative structure to the file.
  **CN L566:** 这一行为文件补充了实现细节或声明式结构。
- **EN L567:** Blank line used to separate nearby declarations and improve readability.
  **CN L567:** 该空行用于分隔相邻声明并提升可读性。
- **EN L568:** This line contributes implementation detail or declarative structure to the file.
  **CN L568:** 这一行为文件补充了实现细节或声明式结构。
- **EN L569:** Blank line used to separate nearby declarations and improve readability.
  **CN L569:** 该空行用于分隔相邻声明并提升可读性。
- **EN L570:** This line contributes implementation detail or declarative structure to the file.
  **CN L570:** 这一行为文件补充了实现细节或声明式结构。
- **EN L571:** This comment states: “Scalar sine and cosine values.”, documenting the intent of the surrounding code.
  **CN L571:** 该注释写道：“Scalar sine and cosine values.”，用于说明周围代码的意图。
- **EN L572:** This line contributes implementation detail or declarative structure to the file.
  **CN L572:** 这一行为文件补充了实现细节或声明式结构。
- **EN L573:** This line contributes implementation detail or declarative structure to the file.
  **CN L573:** 这一行为文件补充了实现细节或声明式结构。
- **EN L574:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L574:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L575:** Blank line used to separate nearby declarations and improve readability.
  **CN L575:** 该空行用于分隔相邻声明并提升可读性。
- **EN L576:** This line contributes implementation detail or declarative structure to the file.
  **CN L576:** 这一行为文件补充了实现细节或声明式结构。

### Lines 577-588 / 第 577-588 行

```tablegen
 577:       DefaultValuedAttr<Arith_FastMathAttr,
 578:                         "::mlir::arith::FastMathFlags::none">:$fastmath);
 579:   let results = (outs FloatLike:$sin, FloatLike:$cos);
 580: 
 581:   let assemblyFormat = [{ $operand (`fastmath` `` $fastmath^)?
 582:                           attr-dict `:` type($operand) }];
 583: 
 584:   let extraClassDeclaration = [{
 585:     std::optional<SmallVector<int64_t, 4>> getShapeForUnroll();
 586:   }];
 587: 
 588:   let hasFolder = 1;
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L577:** This line contributes implementation detail or declarative structure to the file.
  **CN L577:** 这一行为文件补充了实现细节或声明式结构。
- **EN L578:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L578:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L579:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L579:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L580:** Blank line used to separate nearby declarations and improve readability.
  **CN L580:** 该空行用于分隔相邻声明并提升可读性。
- **EN L581:** This line contributes to the declaration or call of `operand`.
  **CN L581:** 这一行为 `operand` 的声明或调用提供内容。
- **EN L582:** This line contributes to the declaration or call of `type`.
  **CN L582:** 这一行为 `type` 的声明或调用提供内容。
- **EN L583:** Blank line used to separate nearby declarations and improve readability.
  **CN L583:** 该空行用于分隔相邻声明并提升可读性。
- **EN L584:** This line contributes implementation detail or declarative structure to the file.
  **CN L584:** 这一行为文件补充了实现细节或声明式结构。
- **EN L585:** This line contributes to the declaration or call of `getShapeForUnroll`.
  **CN L585:** 这一行为 `getShapeForUnroll` 的声明或调用提供内容。
- **EN L586:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L586:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L587:** Blank line used to separate nearby declarations and improve readability.
  **CN L587:** 该空行用于分隔相邻声明并提升可读性。
- **EN L588:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L588:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 589-600 / 第 589-600 行

```tablegen
 589: }
 590: 
 591: //===----------------------------------------------------------------------===//
 592: // CountLeadingZerosOp
 593: //===----------------------------------------------------------------------===//
 594: 
 595: def Math_CountLeadingZerosOp : Math_IntegerUnaryOp<"ctlz"> {
 596:   let summary = "counts the leading zeros an integer value";
 597:   let description = [{
 598:     The `ctlz` operation computes the number of leading zeros of an integer value.
 599:     It operates on scalar, tensor or vector.
 600: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L589:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L589:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L590:** Blank line used to separate nearby declarations and improve readability.
  **CN L590:** 该空行用于分隔相邻声明并提升可读性。
- **EN L591:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L591:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L592:** This comment states: “CountLeadingZerosOp”, documenting the intent of the surrounding code.
  **CN L592:** 该注释写道：“CountLeadingZerosOp”，用于说明周围代码的意图。
- **EN L593:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L593:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L594:** Blank line used to separate nearby declarations and improve readability.
  **CN L594:** 该空行用于分隔相邻声明并提升可读性。
- **EN L595:** This TableGen `def` record introduces `Math_CountLeadingZerosOp`, which later participates in generated MLIR code.
  **CN L595:** 该 TableGen `def` 记录引入了 `Math_CountLeadingZerosOp`，后续会参与生成的 MLIR 代码。
- **EN L596:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L596:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L597:** This line contributes implementation detail or declarative structure to the file.
  **CN L597:** 这一行为文件补充了实现细节或声明式结构。
- **EN L598:** This line contributes implementation detail or declarative structure to the file.
  **CN L598:** 这一行为文件补充了实现细节或声明式结构。
- **EN L599:** This line contributes implementation detail or declarative structure to the file.
  **CN L599:** 这一行为文件补充了实现细节或声明式结构。
- **EN L600:** Blank line used to separate nearby declarations and improve readability.
  **CN L600:** 该空行用于分隔相邻声明并提升可读性。

### Lines 601-612 / 第 601-612 行

```tablegen
 601:     Example:
 602: 
 603:     ```mlir
 604:     // Scalar ctlz function value.
 605:     %a = math.ctlz %b : i32
 606:     ```
 607:   }];
 608:   let hasFolder = 1;
 609: }
 610: 
 611: //===----------------------------------------------------------------------===//
 612: // CountTrailingZerosOp
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L601:** This line contributes implementation detail or declarative structure to the file.
  **CN L601:** 这一行为文件补充了实现细节或声明式结构。
- **EN L602:** Blank line used to separate nearby declarations and improve readability.
  **CN L602:** 该空行用于分隔相邻声明并提升可读性。
- **EN L603:** This line contributes implementation detail or declarative structure to the file.
  **CN L603:** 这一行为文件补充了实现细节或声明式结构。
- **EN L604:** This comment states: “Scalar ctlz function value.”, documenting the intent of the surrounding code.
  **CN L604:** 该注释写道：“Scalar ctlz function value.”，用于说明周围代码的意图。
- **EN L605:** This line contributes implementation detail or declarative structure to the file.
  **CN L605:** 这一行为文件补充了实现细节或声明式结构。
- **EN L606:** This line contributes implementation detail or declarative structure to the file.
  **CN L606:** 这一行为文件补充了实现细节或声明式结构。
- **EN L607:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L607:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L608:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L608:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L609:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L609:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L610:** Blank line used to separate nearby declarations and improve readability.
  **CN L610:** 该空行用于分隔相邻声明并提升可读性。
- **EN L611:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L611:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L612:** This comment states: “CountTrailingZerosOp”, documenting the intent of the surrounding code.
  **CN L612:** 该注释写道：“CountTrailingZerosOp”，用于说明周围代码的意图。

### Lines 613-624 / 第 613-624 行

```tablegen
 613: //===----------------------------------------------------------------------===//
 614: 
 615: def Math_CountTrailingZerosOp : Math_IntegerUnaryOp<"cttz"> {
 616:   let summary = "counts the trailing zeros an integer value";
 617:   let description = [{
 618:     The `cttz` operation computes the number of trailing zeros of an integer value.
 619:     It operates on scalar, tensor or vector.
 620: 
 621:     Example:
 622: 
 623:     ```mlir
 624:     // Scalar cttz function value.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L613:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L613:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L614:** Blank line used to separate nearby declarations and improve readability.
  **CN L614:** 该空行用于分隔相邻声明并提升可读性。
- **EN L615:** This TableGen `def` record introduces `Math_CountTrailingZerosOp`, which later participates in generated MLIR code.
  **CN L615:** 该 TableGen `def` 记录引入了 `Math_CountTrailingZerosOp`，后续会参与生成的 MLIR 代码。
- **EN L616:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L616:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L617:** This line contributes implementation detail or declarative structure to the file.
  **CN L617:** 这一行为文件补充了实现细节或声明式结构。
- **EN L618:** This line contributes implementation detail or declarative structure to the file.
  **CN L618:** 这一行为文件补充了实现细节或声明式结构。
- **EN L619:** This line contributes implementation detail or declarative structure to the file.
  **CN L619:** 这一行为文件补充了实现细节或声明式结构。
- **EN L620:** Blank line used to separate nearby declarations and improve readability.
  **CN L620:** 该空行用于分隔相邻声明并提升可读性。
- **EN L621:** This line contributes implementation detail or declarative structure to the file.
  **CN L621:** 这一行为文件补充了实现细节或声明式结构。
- **EN L622:** Blank line used to separate nearby declarations and improve readability.
  **CN L622:** 该空行用于分隔相邻声明并提升可读性。
- **EN L623:** This line contributes implementation detail or declarative structure to the file.
  **CN L623:** 这一行为文件补充了实现细节或声明式结构。
- **EN L624:** This comment states: “Scalar cttz function value.”, documenting the intent of the surrounding code.
  **CN L624:** 该注释写道：“Scalar cttz function value.”，用于说明周围代码的意图。

### Lines 625-636 / 第 625-636 行

```tablegen
 625:     %a = math.cttz %b : i32
 626:     ```
 627:   }];
 628:   let hasFolder = 1;
 629: }
 630: 
 631: //===----------------------------------------------------------------------===//
 632: // CtPopOp
 633: //===----------------------------------------------------------------------===//
 634: 
 635: def Math_CtPopOp : Math_IntegerUnaryOp<"ctpop"> {
 636:   let summary = "counts the number of set bits of an integer value";
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L625:** This line contributes implementation detail or declarative structure to the file.
  **CN L625:** 这一行为文件补充了实现细节或声明式结构。
- **EN L626:** This line contributes implementation detail or declarative structure to the file.
  **CN L626:** 这一行为文件补充了实现细节或声明式结构。
- **EN L627:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L627:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L628:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L628:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L629:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L629:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L630:** Blank line used to separate nearby declarations and improve readability.
  **CN L630:** 该空行用于分隔相邻声明并提升可读性。
- **EN L631:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L631:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L632:** This comment states: “CtPopOp”, documenting the intent of the surrounding code.
  **CN L632:** 该注释写道：“CtPopOp”，用于说明周围代码的意图。
- **EN L633:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L633:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L634:** Blank line used to separate nearby declarations and improve readability.
  **CN L634:** 该空行用于分隔相邻声明并提升可读性。
- **EN L635:** This TableGen `def` record introduces `Math_CtPopOp`, which later participates in generated MLIR code.
  **CN L635:** 该 TableGen `def` 记录引入了 `Math_CtPopOp`，后续会参与生成的 MLIR 代码。
- **EN L636:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L636:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 637-648 / 第 637-648 行

```tablegen
 637:   let description = [{
 638:     The `ctpop` operation computes the number of set bits of an integer value.
 639:     It operates on scalar, tensor or vector.
 640: 
 641:     Example:
 642: 
 643:     ```mlir
 644:     // Scalar ctpop function value.
 645:     %a = math.ctpop %b : i32
 646:     ```
 647:   }];
 648:   let hasFolder = 1;
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L637:** This line contributes implementation detail or declarative structure to the file.
  **CN L637:** 这一行为文件补充了实现细节或声明式结构。
- **EN L638:** This line contributes implementation detail or declarative structure to the file.
  **CN L638:** 这一行为文件补充了实现细节或声明式结构。
- **EN L639:** This line contributes implementation detail or declarative structure to the file.
  **CN L639:** 这一行为文件补充了实现细节或声明式结构。
- **EN L640:** Blank line used to separate nearby declarations and improve readability.
  **CN L640:** 该空行用于分隔相邻声明并提升可读性。
- **EN L641:** This line contributes implementation detail or declarative structure to the file.
  **CN L641:** 这一行为文件补充了实现细节或声明式结构。
- **EN L642:** Blank line used to separate nearby declarations and improve readability.
  **CN L642:** 该空行用于分隔相邻声明并提升可读性。
- **EN L643:** This line contributes implementation detail or declarative structure to the file.
  **CN L643:** 这一行为文件补充了实现细节或声明式结构。
- **EN L644:** This comment states: “Scalar ctpop function value.”, documenting the intent of the surrounding code.
  **CN L644:** 该注释写道：“Scalar ctpop function value.”，用于说明周围代码的意图。
- **EN L645:** This line contributes implementation detail or declarative structure to the file.
  **CN L645:** 这一行为文件补充了实现细节或声明式结构。
- **EN L646:** This line contributes implementation detail or declarative structure to the file.
  **CN L646:** 这一行为文件补充了实现细节或声明式结构。
- **EN L647:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L647:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L648:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L648:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 649-660 / 第 649-660 行

```tablegen
 649: }
 650: 
 651: //===----------------------------------------------------------------------===//
 652: // ErfOp
 653: //===----------------------------------------------------------------------===//
 654: 
 655: def Math_ErfOp : Math_FloatUnaryOp<"erf"> {
 656:   let summary = "error function of the specified value";
 657:   let description = [{
 658:     The `erf` operation computes the error function. It takes one operand of
 659:     floating point type (i.e., scalar, tensor or vector) and returns one result of
 660:     the same type. It has no standard attributes.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L649:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L649:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L650:** Blank line used to separate nearby declarations and improve readability.
  **CN L650:** 该空行用于分隔相邻声明并提升可读性。
- **EN L651:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L651:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L652:** This comment states: “ErfOp”, documenting the intent of the surrounding code.
  **CN L652:** 该注释写道：“ErfOp”，用于说明周围代码的意图。
- **EN L653:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L653:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L654:** Blank line used to separate nearby declarations and improve readability.
  **CN L654:** 该空行用于分隔相邻声明并提升可读性。
- **EN L655:** This TableGen `def` record introduces `Math_ErfOp`, which later participates in generated MLIR code.
  **CN L655:** 该 TableGen `def` 记录引入了 `Math_ErfOp`，后续会参与生成的 MLIR 代码。
- **EN L656:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L656:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L657:** This line contributes implementation detail or declarative structure to the file.
  **CN L657:** 这一行为文件补充了实现细节或声明式结构。
- **EN L658:** This line contributes implementation detail or declarative structure to the file.
  **CN L658:** 这一行为文件补充了实现细节或声明式结构。
- **EN L659:** This line contributes to the declaration or call of `type`.
  **CN L659:** 这一行为 `type` 的声明或调用提供内容。
- **EN L660:** This line contributes implementation detail or declarative structure to the file.
  **CN L660:** 这一行为文件补充了实现细节或声明式结构。

### Lines 661-672 / 第 661-672 行

```tablegen
 661: 
 662:     Example:
 663: 
 664:     ```mlir
 665:     // Scalar error function value.
 666:     %a = math.erf %b : f64
 667:     ```
 668:   }];
 669:   let hasFolder = 1;
 670: }
 671: 
 672: //===----------------------------------------------------------------------===//
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L661:** Blank line used to separate nearby declarations and improve readability.
  **CN L661:** 该空行用于分隔相邻声明并提升可读性。
- **EN L662:** This line contributes implementation detail or declarative structure to the file.
  **CN L662:** 这一行为文件补充了实现细节或声明式结构。
- **EN L663:** Blank line used to separate nearby declarations and improve readability.
  **CN L663:** 该空行用于分隔相邻声明并提升可读性。
- **EN L664:** This line contributes implementation detail or declarative structure to the file.
  **CN L664:** 这一行为文件补充了实现细节或声明式结构。
- **EN L665:** This comment states: “Scalar error function value.”, documenting the intent of the surrounding code.
  **CN L665:** 该注释写道：“Scalar error function value.”，用于说明周围代码的意图。
- **EN L666:** This line contributes implementation detail or declarative structure to the file.
  **CN L666:** 这一行为文件补充了实现细节或声明式结构。
- **EN L667:** This line contributes implementation detail or declarative structure to the file.
  **CN L667:** 这一行为文件补充了实现细节或声明式结构。
- **EN L668:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L668:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L669:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L669:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L670:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L670:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L671:** Blank line used to separate nearby declarations and improve readability.
  **CN L671:** 该空行用于分隔相邻声明并提升可读性。
- **EN L672:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L672:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 673-684 / 第 673-684 行

```tablegen
 673: // ErfcOp
 674: //===----------------------------------------------------------------------===//
 675: 
 676: def Math_ErfcOp : Math_FloatUnaryOp<"erfc"> {
 677:   let summary = "complementary error function of the specified value";
 678:   let description = [{
 679: 
 680:     The `erfc` operation computes the complementary error function, defined as
 681:     1-erf(x). This function is part of libm and is needed for accuracy, since
 682:     simply calculating 1-erf(x) when x is close to 1 will give inaccurate results.
 683:     It takes one operand of floating point type (i.e., scalar,
 684:     tensor or vector) and returns one result of the same type. It has no
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L673:** This comment states: “ErfcOp”, documenting the intent of the surrounding code.
  **CN L673:** 该注释写道：“ErfcOp”，用于说明周围代码的意图。
- **EN L674:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L674:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L675:** Blank line used to separate nearby declarations and improve readability.
  **CN L675:** 该空行用于分隔相邻声明并提升可读性。
- **EN L676:** This TableGen `def` record introduces `Math_ErfcOp`, which later participates in generated MLIR code.
  **CN L676:** 该 TableGen `def` 记录引入了 `Math_ErfcOp`，后续会参与生成的 MLIR 代码。
- **EN L677:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L677:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L678:** This line contributes implementation detail or declarative structure to the file.
  **CN L678:** 这一行为文件补充了实现细节或声明式结构。
- **EN L679:** Blank line used to separate nearby declarations and improve readability.
  **CN L679:** 该空行用于分隔相邻声明并提升可读性。
- **EN L680:** This line contributes implementation detail or declarative structure to the file.
  **CN L680:** 这一行为文件补充了实现细节或声明式结构。
- **EN L681:** This line contributes to the declaration or call of `erf`.
  **CN L681:** 这一行为 `erf` 的声明或调用提供内容。
- **EN L682:** This line contributes to the declaration or call of `erf`.
  **CN L682:** 这一行为 `erf` 的声明或调用提供内容。
- **EN L683:** This line contributes to the declaration or call of `type`.
  **CN L683:** 这一行为 `type` 的声明或调用提供内容。
- **EN L684:** This line contributes implementation detail or declarative structure to the file.
  **CN L684:** 这一行为文件补充了实现细节或声明式结构。

### Lines 685-696 / 第 685-696 行

```tablegen
 685:     standard attributes.
 686: 
 687:     Example:
 688: 
 689:     ```mlir
 690:     // Scalar error function value.
 691:     %a = math.erfc %b : f64
 692:     ```
 693:   }];
 694:   let hasFolder = 1;
 695: }
 696: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L685:** This line contributes implementation detail or declarative structure to the file.
  **CN L685:** 这一行为文件补充了实现细节或声明式结构。
- **EN L686:** Blank line used to separate nearby declarations and improve readability.
  **CN L686:** 该空行用于分隔相邻声明并提升可读性。
- **EN L687:** This line contributes implementation detail or declarative structure to the file.
  **CN L687:** 这一行为文件补充了实现细节或声明式结构。
- **EN L688:** Blank line used to separate nearby declarations and improve readability.
  **CN L688:** 该空行用于分隔相邻声明并提升可读性。
- **EN L689:** This line contributes implementation detail or declarative structure to the file.
  **CN L689:** 这一行为文件补充了实现细节或声明式结构。
- **EN L690:** This comment states: “Scalar error function value.”, documenting the intent of the surrounding code.
  **CN L690:** 该注释写道：“Scalar error function value.”，用于说明周围代码的意图。
- **EN L691:** This line contributes implementation detail or declarative structure to the file.
  **CN L691:** 这一行为文件补充了实现细节或声明式结构。
- **EN L692:** This line contributes implementation detail or declarative structure to the file.
  **CN L692:** 这一行为文件补充了实现细节或声明式结构。
- **EN L693:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L693:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L694:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L694:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L695:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L695:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L696:** Blank line used to separate nearby declarations and improve readability.
  **CN L696:** 该空行用于分隔相邻声明并提升可读性。

### Lines 697-708 / 第 697-708 行

```tablegen
 697: 
 698: //===----------------------------------------------------------------------===//
 699: // ExpOp
 700: //===----------------------------------------------------------------------===//
 701: 
 702: def Math_ExpOp : Math_FloatUnaryOp<"exp"> {
 703:   let summary = "base-e exponential of the specified value";
 704:   let description = [{
 705:     The `exp` operation takes one operand of floating point type (i.e., scalar,
 706:     tensor or vector) and returns one result of the same type. It has no standard
 707:     attributes.
 708: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L697:** Blank line used to separate nearby declarations and improve readability.
  **CN L697:** 该空行用于分隔相邻声明并提升可读性。
- **EN L698:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L698:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L699:** This comment states: “ExpOp”, documenting the intent of the surrounding code.
  **CN L699:** 该注释写道：“ExpOp”，用于说明周围代码的意图。
- **EN L700:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L700:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L701:** Blank line used to separate nearby declarations and improve readability.
  **CN L701:** 该空行用于分隔相邻声明并提升可读性。
- **EN L702:** This TableGen `def` record introduces `Math_ExpOp`, which later participates in generated MLIR code.
  **CN L702:** 该 TableGen `def` 记录引入了 `Math_ExpOp`，后续会参与生成的 MLIR 代码。
- **EN L703:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L703:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L704:** This line contributes implementation detail or declarative structure to the file.
  **CN L704:** 这一行为文件补充了实现细节或声明式结构。
- **EN L705:** This line contributes to the declaration or call of `type`.
  **CN L705:** 这一行为 `type` 的声明或调用提供内容。
- **EN L706:** This line contributes implementation detail or declarative structure to the file.
  **CN L706:** 这一行为文件补充了实现细节或声明式结构。
- **EN L707:** This line contributes implementation detail or declarative structure to the file.
  **CN L707:** 这一行为文件补充了实现细节或声明式结构。
- **EN L708:** Blank line used to separate nearby declarations and improve readability.
  **CN L708:** 该空行用于分隔相邻声明并提升可读性。

### Lines 709-720 / 第 709-720 行

```tablegen
 709:     Example:
 710: 
 711:     ```mlir
 712:     // Scalar natural exponential.
 713:     %a = math.exp %b : f64
 714:     ```
 715:   }];
 716:   let hasFolder = 1;
 717: }
 718: 
 719: //===----------------------------------------------------------------------===//
 720: // Exp2Op
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L709:** This line contributes implementation detail or declarative structure to the file.
  **CN L709:** 这一行为文件补充了实现细节或声明式结构。
- **EN L710:** Blank line used to separate nearby declarations and improve readability.
  **CN L710:** 该空行用于分隔相邻声明并提升可读性。
- **EN L711:** This line contributes implementation detail or declarative structure to the file.
  **CN L711:** 这一行为文件补充了实现细节或声明式结构。
- **EN L712:** This comment states: “Scalar natural exponential.”, documenting the intent of the surrounding code.
  **CN L712:** 该注释写道：“Scalar natural exponential.”，用于说明周围代码的意图。
- **EN L713:** This line contributes implementation detail or declarative structure to the file.
  **CN L713:** 这一行为文件补充了实现细节或声明式结构。
- **EN L714:** This line contributes implementation detail or declarative structure to the file.
  **CN L714:** 这一行为文件补充了实现细节或声明式结构。
- **EN L715:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L715:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L716:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L716:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L717:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L717:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L718:** Blank line used to separate nearby declarations and improve readability.
  **CN L718:** 该空行用于分隔相邻声明并提升可读性。
- **EN L719:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L719:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L720:** This comment states: “Exp2Op”, documenting the intent of the surrounding code.
  **CN L720:** 该注释写道：“Exp2Op”，用于说明周围代码的意图。

### Lines 721-732 / 第 721-732 行

```tablegen
 721: //===----------------------------------------------------------------------===//
 722: 
 723: def Math_Exp2Op : Math_FloatUnaryOp<"exp2"> {
 724:   let summary = "base-2 exponential of the specified value";
 725: 
 726:   let description = [{
 727:     The `exp` operation takes one operand of floating point type (i.e., scalar,
 728:     tensor or vector) and returns one result of the same type. It has no standard
 729:     attributes.
 730: 
 731:     Example:
 732: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L721:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L721:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L722:** Blank line used to separate nearby declarations and improve readability.
  **CN L722:** 该空行用于分隔相邻声明并提升可读性。
- **EN L723:** This TableGen `def` record introduces `Math_Exp2Op`, which later participates in generated MLIR code.
  **CN L723:** 该 TableGen `def` 记录引入了 `Math_Exp2Op`，后续会参与生成的 MLIR 代码。
- **EN L724:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L724:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L725:** Blank line used to separate nearby declarations and improve readability.
  **CN L725:** 该空行用于分隔相邻声明并提升可读性。
- **EN L726:** This line contributes implementation detail or declarative structure to the file.
  **CN L726:** 这一行为文件补充了实现细节或声明式结构。
- **EN L727:** This line contributes to the declaration or call of `type`.
  **CN L727:** 这一行为 `type` 的声明或调用提供内容。
- **EN L728:** This line contributes implementation detail or declarative structure to the file.
  **CN L728:** 这一行为文件补充了实现细节或声明式结构。
- **EN L729:** This line contributes implementation detail or declarative structure to the file.
  **CN L729:** 这一行为文件补充了实现细节或声明式结构。
- **EN L730:** Blank line used to separate nearby declarations and improve readability.
  **CN L730:** 该空行用于分隔相邻声明并提升可读性。
- **EN L731:** This line contributes implementation detail or declarative structure to the file.
  **CN L731:** 这一行为文件补充了实现细节或声明式结构。
- **EN L732:** Blank line used to separate nearby declarations and improve readability.
  **CN L732:** 该空行用于分隔相邻声明并提升可读性。

### Lines 733-744 / 第 733-744 行

```tablegen
 733:     ```mlir
 734:     // Scalar natural exponential.
 735:     %a = math.exp2 %b : f64
 736:     ```
 737:   }];
 738:   let hasFolder = 1;
 739: }
 740: 
 741: //===----------------------------------------------------------------------===//
 742: // ExpM1Op
 743: //===----------------------------------------------------------------------===//
 744: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L733:** This line contributes implementation detail or declarative structure to the file.
  **CN L733:** 这一行为文件补充了实现细节或声明式结构。
- **EN L734:** This comment states: “Scalar natural exponential.”, documenting the intent of the surrounding code.
  **CN L734:** 该注释写道：“Scalar natural exponential.”，用于说明周围代码的意图。
- **EN L735:** This line contributes implementation detail or declarative structure to the file.
  **CN L735:** 这一行为文件补充了实现细节或声明式结构。
- **EN L736:** This line contributes implementation detail or declarative structure to the file.
  **CN L736:** 这一行为文件补充了实现细节或声明式结构。
- **EN L737:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L737:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L738:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L738:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L739:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L739:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L740:** Blank line used to separate nearby declarations and improve readability.
  **CN L740:** 该空行用于分隔相邻声明并提升可读性。
- **EN L741:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L741:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L742:** This comment states: “ExpM1Op”, documenting the intent of the surrounding code.
  **CN L742:** 该注释写道：“ExpM1Op”，用于说明周围代码的意图。
- **EN L743:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L743:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L744:** Blank line used to separate nearby declarations and improve readability.
  **CN L744:** 该空行用于分隔相邻声明并提升可读性。

### Lines 745-756 / 第 745-756 行

```tablegen
 745: def Math_ExpM1Op : Math_FloatUnaryOp<"expm1"> {
 746:   let summary = "base-e exponential of the specified value minus 1";
 747:   let description = [{
 748:     expm1(x) := exp(x) - 1
 749: 
 750:     The `expm1` operation takes one operand of floating point type (i.e.,
 751:     scalar, tensor or vector) and returns one result of the same type. It has no
 752:     standard attributes.
 753: 
 754:     Example:
 755: 
 756:     ```mlir
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L745:** This TableGen `def` record introduces `Math_ExpM1Op`, which later participates in generated MLIR code.
  **CN L745:** 该 TableGen `def` 记录引入了 `Math_ExpM1Op`，后续会参与生成的 MLIR 代码。
- **EN L746:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L746:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L747:** This line contributes implementation detail or declarative structure to the file.
  **CN L747:** 这一行为文件补充了实现细节或声明式结构。
- **EN L748:** This line contributes to the declaration or call of `expm1`.
  **CN L748:** 这一行为 `expm1` 的声明或调用提供内容。
- **EN L749:** Blank line used to separate nearby declarations and improve readability.
  **CN L749:** 该空行用于分隔相邻声明并提升可读性。
- **EN L750:** This line contributes to the declaration or call of `type`.
  **CN L750:** 这一行为 `type` 的声明或调用提供内容。
- **EN L751:** This line contributes implementation detail or declarative structure to the file.
  **CN L751:** 这一行为文件补充了实现细节或声明式结构。
- **EN L752:** This line contributes implementation detail or declarative structure to the file.
  **CN L752:** 这一行为文件补充了实现细节或声明式结构。
- **EN L753:** Blank line used to separate nearby declarations and improve readability.
  **CN L753:** 该空行用于分隔相邻声明并提升可读性。
- **EN L754:** This line contributes implementation detail or declarative structure to the file.
  **CN L754:** 这一行为文件补充了实现细节或声明式结构。
- **EN L755:** Blank line used to separate nearby declarations and improve readability.
  **CN L755:** 该空行用于分隔相邻声明并提升可读性。
- **EN L756:** This line contributes implementation detail or declarative structure to the file.
  **CN L756:** 这一行为文件补充了实现细节或声明式结构。

### Lines 757-768 / 第 757-768 行

```tablegen
 757:     // Scalar natural exponential minus 1.
 758:     %a = math.expm1 %b : f64
 759:     ```
 760:   }];
 761:   let hasFolder = 1;
 762: }
 763: 
 764: //===----------------------------------------------------------------------===//
 765: // FloorOp
 766: //===----------------------------------------------------------------------===//
 767: 
 768: def Math_FloorOp : Math_FloatUnaryOp<"floor"> {
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L757:** This comment states: “Scalar natural exponential minus 1.”, documenting the intent of the surrounding code.
  **CN L757:** 该注释写道：“Scalar natural exponential minus 1.”，用于说明周围代码的意图。
- **EN L758:** This line contributes implementation detail or declarative structure to the file.
  **CN L758:** 这一行为文件补充了实现细节或声明式结构。
- **EN L759:** This line contributes implementation detail or declarative structure to the file.
  **CN L759:** 这一行为文件补充了实现细节或声明式结构。
- **EN L760:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L760:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L761:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L761:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L762:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L762:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L763:** Blank line used to separate nearby declarations and improve readability.
  **CN L763:** 该空行用于分隔相邻声明并提升可读性。
- **EN L764:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L764:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L765:** This comment states: “FloorOp”, documenting the intent of the surrounding code.
  **CN L765:** 该注释写道：“FloorOp”，用于说明周围代码的意图。
- **EN L766:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L766:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L767:** Blank line used to separate nearby declarations and improve readability.
  **CN L767:** 该空行用于分隔相邻声明并提升可读性。
- **EN L768:** This TableGen `def` record introduces `Math_FloorOp`, which later participates in generated MLIR code.
  **CN L768:** 该 TableGen `def` 记录引入了 `Math_FloorOp`，后续会参与生成的 MLIR 代码。

### Lines 769-780 / 第 769-780 行

```tablegen
 769:   let summary = "floor of the specified value";
 770:   let description = [{
 771:     The `floor` operation computes the floor of a given value. It takes one
 772:     operand of floating point type (i.e., scalar, tensor or vector) and returns one
 773:     result of the same type.  It has no standard attributes.
 774: 
 775:     Example:
 776: 
 777:     ```mlir
 778:     // Scalar floor value.
 779:     %a = math.floor %b : f64
 780:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L769:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L769:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L770:** This line contributes implementation detail or declarative structure to the file.
  **CN L770:** 这一行为文件补充了实现细节或声明式结构。
- **EN L771:** This line contributes implementation detail or declarative structure to the file.
  **CN L771:** 这一行为文件补充了实现细节或声明式结构。
- **EN L772:** This line contributes to the declaration or call of `type`.
  **CN L772:** 这一行为 `type` 的声明或调用提供内容。
- **EN L773:** This line contributes implementation detail or declarative structure to the file.
  **CN L773:** 这一行为文件补充了实现细节或声明式结构。
- **EN L774:** Blank line used to separate nearby declarations and improve readability.
  **CN L774:** 该空行用于分隔相邻声明并提升可读性。
- **EN L775:** This line contributes implementation detail or declarative structure to the file.
  **CN L775:** 这一行为文件补充了实现细节或声明式结构。
- **EN L776:** Blank line used to separate nearby declarations and improve readability.
  **CN L776:** 该空行用于分隔相邻声明并提升可读性。
- **EN L777:** This line contributes implementation detail or declarative structure to the file.
  **CN L777:** 这一行为文件补充了实现细节或声明式结构。
- **EN L778:** This comment states: “Scalar floor value.”, documenting the intent of the surrounding code.
  **CN L778:** 该注释写道：“Scalar floor value.”，用于说明周围代码的意图。
- **EN L779:** This line contributes implementation detail or declarative structure to the file.
  **CN L779:** 这一行为文件补充了实现细节或声明式结构。
- **EN L780:** This line contributes implementation detail or declarative structure to the file.
  **CN L780:** 这一行为文件补充了实现细节或声明式结构。

### Lines 781-792 / 第 781-792 行

```tablegen
 781:   }];
 782: 
 783:   let hasFolder = 1;
 784: }
 785: 
 786: //===----------------------------------------------------------------------===//
 787: // FmaOp
 788: //===----------------------------------------------------------------------===//
 789: 
 790: def Math_FmaOp : Math_FloatTernaryOpWithRoundingMode<"fma"> {
 791:   let summary = "floating point fused multipy-add operation";
 792:   let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L781:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L781:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L782:** Blank line used to separate nearby declarations and improve readability.
  **CN L782:** 该空行用于分隔相邻声明并提升可读性。
- **EN L783:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L783:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L784:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L784:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L785:** Blank line used to separate nearby declarations and improve readability.
  **CN L785:** 该空行用于分隔相邻声明并提升可读性。
- **EN L786:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L786:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L787:** This comment states: “FmaOp”, documenting the intent of the surrounding code.
  **CN L787:** 该注释写道：“FmaOp”，用于说明周围代码的意图。
- **EN L788:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L788:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L789:** Blank line used to separate nearby declarations and improve readability.
  **CN L789:** 该空行用于分隔相邻声明并提升可读性。
- **EN L790:** This TableGen `def` record introduces `Math_FmaOp`, which later participates in generated MLIR code.
  **CN L790:** 该 TableGen `def` 记录引入了 `Math_FmaOp`，后续会参与生成的 MLIR 代码。
- **EN L791:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L791:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L792:** This line contributes implementation detail or declarative structure to the file.
  **CN L792:** 这一行为文件补充了实现细节或声明式结构。

### Lines 793-804 / 第 793-804 行

```tablegen
 793:     The `fma` operation takes three operands and returns one result, each of
 794:     these is required to be the same type. Operands must be of floating point type
 795:     (i.e., scalar, tensor or vector).
 796: 
 797:     Example:
 798: 
 799:     ```mlir
 800:     // Scalar fused multiply-add: d = a*b + c
 801:     %d = math.fma %a, %b, %c : f64
 802: 
 803:     // With an explicit IEEE-754 rounding mode.
 804:     %e = math.fma %a, %b, %c to_nearest_even : f64
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L793:** This line contributes implementation detail or declarative structure to the file.
  **CN L793:** 这一行为文件补充了实现细节或声明式结构。
- **EN L794:** This line contributes implementation detail or declarative structure to the file.
  **CN L794:** 这一行为文件补充了实现细节或声明式结构。
- **EN L795:** This line contributes implementation detail or declarative structure to the file.
  **CN L795:** 这一行为文件补充了实现细节或声明式结构。
- **EN L796:** Blank line used to separate nearby declarations and improve readability.
  **CN L796:** 该空行用于分隔相邻声明并提升可读性。
- **EN L797:** This line contributes implementation detail or declarative structure to the file.
  **CN L797:** 这一行为文件补充了实现细节或声明式结构。
- **EN L798:** Blank line used to separate nearby declarations and improve readability.
  **CN L798:** 该空行用于分隔相邻声明并提升可读性。
- **EN L799:** This line contributes implementation detail or declarative structure to the file.
  **CN L799:** 这一行为文件补充了实现细节或声明式结构。
- **EN L800:** This comment states: “Scalar fused multiply-add: d = a*b + c”, documenting the intent of the surrounding code.
  **CN L800:** 该注释写道：“Scalar fused multiply-add: d = a*b + c”，用于说明周围代码的意图。
- **EN L801:** This line contributes implementation detail or declarative structure to the file.
  **CN L801:** 这一行为文件补充了实现细节或声明式结构。
- **EN L802:** Blank line used to separate nearby declarations and improve readability.
  **CN L802:** 该空行用于分隔相邻声明并提升可读性。
- **EN L803:** This comment states: “With an explicit IEEE-754 rounding mode.”, documenting the intent of the surrounding code.
  **CN L803:** 该注释写道：“With an explicit IEEE-754 rounding mode.”，用于说明周围代码的意图。
- **EN L804:** This line contributes implementation detail or declarative structure to the file.
  **CN L804:** 这一行为文件补充了实现细节或声明式结构。

### Lines 805-816 / 第 805-816 行

```tablegen
 805:     ```
 806: 
 807:     The semantics of the operation correspond to those of the `llvm.fma`
 808:     [intrinsic](https://llvm.org/docs/LangRef.html#llvm-fma-intrinsic). When
 809:     no rounding mode is set, lowering to LLVM is guaranteed to produce the
 810:     `llvm.fma.*` intrinsic. When a rounding mode is set, the LLVM lowering
 811:     instead produces `llvm.experimental.constrained.fma`.
 812:   }];
 813: }
 814: 
 815: //===----------------------------------------------------------------------===//
 816: // IPowIOp
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L805:** This line contributes implementation detail or declarative structure to the file.
  **CN L805:** 这一行为文件补充了实现细节或声明式结构。
- **EN L806:** Blank line used to separate nearby declarations and improve readability.
  **CN L806:** 该空行用于分隔相邻声明并提升可读性。
- **EN L807:** This line contributes implementation detail or declarative structure to the file.
  **CN L807:** 这一行为文件补充了实现细节或声明式结构。
- **EN L808:** This line contributes implementation detail or declarative structure to the file.
  **CN L808:** 这一行为文件补充了实现细节或声明式结构。
- **EN L809:** This line contributes implementation detail or declarative structure to the file.
  **CN L809:** 这一行为文件补充了实现细节或声明式结构。
- **EN L810:** This line contributes implementation detail or declarative structure to the file.
  **CN L810:** 这一行为文件补充了实现细节或声明式结构。
- **EN L811:** This line contributes implementation detail or declarative structure to the file.
  **CN L811:** 这一行为文件补充了实现细节或声明式结构。
- **EN L812:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L812:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L813:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L813:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L814:** Blank line used to separate nearby declarations and improve readability.
  **CN L814:** 该空行用于分隔相邻声明并提升可读性。
- **EN L815:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L815:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L816:** This comment states: “IPowIOp”, documenting the intent of the surrounding code.
  **CN L816:** 该注释写道：“IPowIOp”，用于说明周围代码的意图。

### Lines 817-828 / 第 817-828 行

```tablegen
 817: //===----------------------------------------------------------------------===//
 818: 
 819: def Math_IPowIOp : Math_IntegerBinaryOp<"ipowi"> {
 820:   let summary = "signed integer raised to the power of operation";
 821:   let description = [{
 822:     The `ipowi` operation takes two operands of integer type (i.e., scalar,
 823:     tensor or vector) and returns one result of the same type. Operands
 824:     must have the same type.
 825: 
 826:     Example:
 827: 
 828:     ```mlir
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L817:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L817:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L818:** Blank line used to separate nearby declarations and improve readability.
  **CN L818:** 该空行用于分隔相邻声明并提升可读性。
- **EN L819:** This TableGen `def` record introduces `Math_IPowIOp`, which later participates in generated MLIR code.
  **CN L819:** 该 TableGen `def` 记录引入了 `Math_IPowIOp`，后续会参与生成的 MLIR 代码。
- **EN L820:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L820:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L821:** This line contributes implementation detail or declarative structure to the file.
  **CN L821:** 这一行为文件补充了实现细节或声明式结构。
- **EN L822:** This line contributes to the declaration or call of `type`.
  **CN L822:** 这一行为 `type` 的声明或调用提供内容。
- **EN L823:** This line contributes implementation detail or declarative structure to the file.
  **CN L823:** 这一行为文件补充了实现细节或声明式结构。
- **EN L824:** This line contributes implementation detail or declarative structure to the file.
  **CN L824:** 这一行为文件补充了实现细节或声明式结构。
- **EN L825:** Blank line used to separate nearby declarations and improve readability.
  **CN L825:** 该空行用于分隔相邻声明并提升可读性。
- **EN L826:** This line contributes implementation detail or declarative structure to the file.
  **CN L826:** 这一行为文件补充了实现细节或声明式结构。
- **EN L827:** Blank line used to separate nearby declarations and improve readability.
  **CN L827:** 该空行用于分隔相邻声明并提升可读性。
- **EN L828:** This line contributes implementation detail or declarative structure to the file.
  **CN L828:** 这一行为文件补充了实现细节或声明式结构。

### Lines 829-840 / 第 829-840 行

```tablegen
 829:     // Scalar signed integer exponentiation.
 830:     %a = math.ipowi %b, %c : i32
 831:     ```
 832:   }];
 833:   let hasFolder = 1;
 834: }
 835: 
 836: //===----------------------------------------------------------------------===//
 837: // IsFiniteOp
 838: //===----------------------------------------------------------------------===//
 839: 
 840: def Math_IsFiniteOp : Math_FloatClassificationOp<"isfinite"> {
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L829:** This comment states: “Scalar signed integer exponentiation.”, documenting the intent of the surrounding code.
  **CN L829:** 该注释写道：“Scalar signed integer exponentiation.”，用于说明周围代码的意图。
- **EN L830:** This line contributes implementation detail or declarative structure to the file.
  **CN L830:** 这一行为文件补充了实现细节或声明式结构。
- **EN L831:** This line contributes implementation detail or declarative structure to the file.
  **CN L831:** 这一行为文件补充了实现细节或声明式结构。
- **EN L832:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L832:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L833:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L833:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L834:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L834:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L835:** Blank line used to separate nearby declarations and improve readability.
  **CN L835:** 该空行用于分隔相邻声明并提升可读性。
- **EN L836:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L836:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L837:** This comment states: “IsFiniteOp”, documenting the intent of the surrounding code.
  **CN L837:** 该注释写道：“IsFiniteOp”，用于说明周围代码的意图。
- **EN L838:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L838:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L839:** Blank line used to separate nearby declarations and improve readability.
  **CN L839:** 该空行用于分隔相邻声明并提升可读性。
- **EN L840:** This TableGen `def` record introduces `Math_IsFiniteOp`, which later participates in generated MLIR code.
  **CN L840:** 该 TableGen `def` 记录引入了 `Math_IsFiniteOp`，后续会参与生成的 MLIR 代码。

### Lines 841-852 / 第 841-852 行

```tablegen
 841:   let summary = "returns true if the operand classifies as finite";
 842:   let description = [{
 843:     Determines if the given floating-point number has finite value i.e. it
 844:     is normal, subnormal or zero, but not infinite or NaN.
 845: 
 846:     Example:
 847: 
 848:     ```mlir
 849:     %f = math.isfinite %a : f32
 850:     ```
 851:   }];
 852:   let hasFolder = 1;
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L841:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L841:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L842:** This line contributes implementation detail or declarative structure to the file.
  **CN L842:** 这一行为文件补充了实现细节或声明式结构。
- **EN L843:** This line contributes implementation detail or declarative structure to the file.
  **CN L843:** 这一行为文件补充了实现细节或声明式结构。
- **EN L844:** This line contributes implementation detail or declarative structure to the file.
  **CN L844:** 这一行为文件补充了实现细节或声明式结构。
- **EN L845:** Blank line used to separate nearby declarations and improve readability.
  **CN L845:** 该空行用于分隔相邻声明并提升可读性。
- **EN L846:** This line contributes implementation detail or declarative structure to the file.
  **CN L846:** 这一行为文件补充了实现细节或声明式结构。
- **EN L847:** Blank line used to separate nearby declarations and improve readability.
  **CN L847:** 该空行用于分隔相邻声明并提升可读性。
- **EN L848:** This line contributes implementation detail or declarative structure to the file.
  **CN L848:** 这一行为文件补充了实现细节或声明式结构。
- **EN L849:** This line contributes implementation detail or declarative structure to the file.
  **CN L849:** 这一行为文件补充了实现细节或声明式结构。
- **EN L850:** This line contributes implementation detail or declarative structure to the file.
  **CN L850:** 这一行为文件补充了实现细节或声明式结构。
- **EN L851:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L851:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L852:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L852:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 853-864 / 第 853-864 行

```tablegen
 853: }
 854: 
 855: //===----------------------------------------------------------------------===//
 856: // IsInfOp
 857: //===----------------------------------------------------------------------===//
 858: 
 859: def Math_IsInfOp : Math_FloatClassificationOp<"isinf"> {
 860:   let summary = "returns true if the operand classifies as infinite";
 861:   let description = [{
 862:     Determines if the given floating-point number is positive or negative
 863:     infinity.
 864: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L853:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L853:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L854:** Blank line used to separate nearby declarations and improve readability.
  **CN L854:** 该空行用于分隔相邻声明并提升可读性。
- **EN L855:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L855:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L856:** This comment states: “IsInfOp”, documenting the intent of the surrounding code.
  **CN L856:** 该注释写道：“IsInfOp”，用于说明周围代码的意图。
- **EN L857:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L857:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L858:** Blank line used to separate nearby declarations and improve readability.
  **CN L858:** 该空行用于分隔相邻声明并提升可读性。
- **EN L859:** This TableGen `def` record introduces `Math_IsInfOp`, which later participates in generated MLIR code.
  **CN L859:** 该 TableGen `def` 记录引入了 `Math_IsInfOp`，后续会参与生成的 MLIR 代码。
- **EN L860:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L860:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L861:** This line contributes implementation detail or declarative structure to the file.
  **CN L861:** 这一行为文件补充了实现细节或声明式结构。
- **EN L862:** This line contributes implementation detail or declarative structure to the file.
  **CN L862:** 这一行为文件补充了实现细节或声明式结构。
- **EN L863:** This line contributes implementation detail or declarative structure to the file.
  **CN L863:** 这一行为文件补充了实现细节或声明式结构。
- **EN L864:** Blank line used to separate nearby declarations and improve readability.
  **CN L864:** 该空行用于分隔相邻声明并提升可读性。

### Lines 865-876 / 第 865-876 行

```tablegen
 865:     Example:
 866: 
 867:     ```mlir
 868:     %f = math.isinf %a : f32
 869:     ```
 870:   }];
 871:   let hasFolder = 1;
 872: }
 873: 
 874: //===----------------------------------------------------------------------===//
 875: // IsNaNOp
 876: //===----------------------------------------------------------------------===//
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L865:** This line contributes implementation detail or declarative structure to the file.
  **CN L865:** 这一行为文件补充了实现细节或声明式结构。
- **EN L866:** Blank line used to separate nearby declarations and improve readability.
  **CN L866:** 该空行用于分隔相邻声明并提升可读性。
- **EN L867:** This line contributes implementation detail or declarative structure to the file.
  **CN L867:** 这一行为文件补充了实现细节或声明式结构。
- **EN L868:** This line contributes implementation detail or declarative structure to the file.
  **CN L868:** 这一行为文件补充了实现细节或声明式结构。
- **EN L869:** This line contributes implementation detail or declarative structure to the file.
  **CN L869:** 这一行为文件补充了实现细节或声明式结构。
- **EN L870:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L870:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L871:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L871:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L872:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L872:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L873:** Blank line used to separate nearby declarations and improve readability.
  **CN L873:** 该空行用于分隔相邻声明并提升可读性。
- **EN L874:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L874:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L875:** This comment states: “IsNaNOp”, documenting the intent of the surrounding code.
  **CN L875:** 该注释写道：“IsNaNOp”，用于说明周围代码的意图。
- **EN L876:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L876:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 877-888 / 第 877-888 行

```tablegen
 877: 
 878: def Math_IsNaNOp : Math_FloatClassificationOp<"isnan"> {
 879:   let summary = "returns true if the operand classifies as NaN";
 880:   let description = [{
 881:     Determines if the given floating-point number is a not-a-number (NaN)
 882:     value.
 883: 
 884:     Example:
 885: 
 886:     ```mlir
 887:     %f = math.isnan %a : f32
 888:     ```
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L877:** Blank line used to separate nearby declarations and improve readability.
  **CN L877:** 该空行用于分隔相邻声明并提升可读性。
- **EN L878:** This TableGen `def` record introduces `Math_IsNaNOp`, which later participates in generated MLIR code.
  **CN L878:** 该 TableGen `def` 记录引入了 `Math_IsNaNOp`，后续会参与生成的 MLIR 代码。
- **EN L879:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L879:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L880:** This line contributes implementation detail or declarative structure to the file.
  **CN L880:** 这一行为文件补充了实现细节或声明式结构。
- **EN L881:** This line contributes to the declaration or call of `number`.
  **CN L881:** 这一行为 `number` 的声明或调用提供内容。
- **EN L882:** This line contributes implementation detail or declarative structure to the file.
  **CN L882:** 这一行为文件补充了实现细节或声明式结构。
- **EN L883:** Blank line used to separate nearby declarations and improve readability.
  **CN L883:** 该空行用于分隔相邻声明并提升可读性。
- **EN L884:** This line contributes implementation detail or declarative structure to the file.
  **CN L884:** 这一行为文件补充了实现细节或声明式结构。
- **EN L885:** Blank line used to separate nearby declarations and improve readability.
  **CN L885:** 该空行用于分隔相邻声明并提升可读性。
- **EN L886:** This line contributes implementation detail or declarative structure to the file.
  **CN L886:** 这一行为文件补充了实现细节或声明式结构。
- **EN L887:** This line contributes implementation detail or declarative structure to the file.
  **CN L887:** 这一行为文件补充了实现细节或声明式结构。
- **EN L888:** This line contributes implementation detail or declarative structure to the file.
  **CN L888:** 这一行为文件补充了实现细节或声明式结构。

### Lines 889-900 / 第 889-900 行

```tablegen
 889:   }];
 890:   let hasFolder = 1;
 891: }
 892: 
 893: 
 894: //===----------------------------------------------------------------------===//
 895: // IsNormalOp
 896: //===----------------------------------------------------------------------===//
 897: 
 898: def Math_IsNormalOp : Math_FloatClassificationOp<"isnormal"> {
 899:   let summary = "returns true if the operand classifies as normal";
 900:   let description = [{
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L889:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L889:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L890:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L890:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L891:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L891:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L892:** Blank line used to separate nearby declarations and improve readability.
  **CN L892:** 该空行用于分隔相邻声明并提升可读性。
- **EN L893:** Blank line used to separate nearby declarations and improve readability.
  **CN L893:** 该空行用于分隔相邻声明并提升可读性。
- **EN L894:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L894:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L895:** This comment states: “IsNormalOp”, documenting the intent of the surrounding code.
  **CN L895:** 该注释写道：“IsNormalOp”，用于说明周围代码的意图。
- **EN L896:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L896:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L897:** Blank line used to separate nearby declarations and improve readability.
  **CN L897:** 该空行用于分隔相邻声明并提升可读性。
- **EN L898:** This TableGen `def` record introduces `Math_IsNormalOp`, which later participates in generated MLIR code.
  **CN L898:** 该 TableGen `def` 记录引入了 `Math_IsNormalOp`，后续会参与生成的 MLIR 代码。
- **EN L899:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L899:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L900:** This line contributes implementation detail or declarative structure to the file.
  **CN L900:** 这一行为文件补充了实现细节或声明式结构。

### Lines 901-912 / 第 901-912 行

```tablegen
 901:     Determines if the given floating-point number is normal, i.e. is neither
 902:     zero, subnormal, infinite, nor NaN.
 903: 
 904:     Example:
 905: 
 906:     ```mlir
 907:     %f = math.isnormal %a : f32
 908:     ```
 909:   }];
 910:   let hasFolder = 1;
 911: }
 912: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L901:** This line contributes implementation detail or declarative structure to the file.
  **CN L901:** 这一行为文件补充了实现细节或声明式结构。
- **EN L902:** This line contributes implementation detail or declarative structure to the file.
  **CN L902:** 这一行为文件补充了实现细节或声明式结构。
- **EN L903:** Blank line used to separate nearby declarations and improve readability.
  **CN L903:** 该空行用于分隔相邻声明并提升可读性。
- **EN L904:** This line contributes implementation detail or declarative structure to the file.
  **CN L904:** 这一行为文件补充了实现细节或声明式结构。
- **EN L905:** Blank line used to separate nearby declarations and improve readability.
  **CN L905:** 该空行用于分隔相邻声明并提升可读性。
- **EN L906:** This line contributes implementation detail or declarative structure to the file.
  **CN L906:** 这一行为文件补充了实现细节或声明式结构。
- **EN L907:** This line contributes implementation detail or declarative structure to the file.
  **CN L907:** 这一行为文件补充了实现细节或声明式结构。
- **EN L908:** This line contributes implementation detail or declarative structure to the file.
  **CN L908:** 这一行为文件补充了实现细节或声明式结构。
- **EN L909:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L909:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L910:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L910:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L911:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L911:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L912:** Blank line used to separate nearby declarations and improve readability.
  **CN L912:** 该空行用于分隔相邻声明并提升可读性。

### Lines 913-924 / 第 913-924 行

```tablegen
 913: //===----------------------------------------------------------------------===//
 914: // LogOp
 915: //===----------------------------------------------------------------------===//
 916: 
 917: def Math_LogOp : Math_FloatUnaryOp<"log"> {
 918:   let summary = "base-e logarithm of the specified value";
 919: 
 920:   let description = [{
 921:     Computes the base-e logarithm of the given value. It takes one operand of
 922:     floating point type (i.e., scalar, tensor or vector) and returns one result of
 923:     the same type.
 924: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L913:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L913:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L914:** This comment states: “LogOp”, documenting the intent of the surrounding code.
  **CN L914:** 该注释写道：“LogOp”，用于说明周围代码的意图。
- **EN L915:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L915:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L916:** Blank line used to separate nearby declarations and improve readability.
  **CN L916:** 该空行用于分隔相邻声明并提升可读性。
- **EN L917:** This TableGen `def` record introduces `Math_LogOp`, which later participates in generated MLIR code.
  **CN L917:** 该 TableGen `def` 记录引入了 `Math_LogOp`，后续会参与生成的 MLIR 代码。
- **EN L918:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L918:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L919:** Blank line used to separate nearby declarations and improve readability.
  **CN L919:** 该空行用于分隔相邻声明并提升可读性。
- **EN L920:** This line contributes implementation detail or declarative structure to the file.
  **CN L920:** 这一行为文件补充了实现细节或声明式结构。
- **EN L921:** This line contributes implementation detail or declarative structure to the file.
  **CN L921:** 这一行为文件补充了实现细节或声明式结构。
- **EN L922:** This line contributes to the declaration or call of `type`.
  **CN L922:** 这一行为 `type` 的声明或调用提供内容。
- **EN L923:** This line contributes implementation detail or declarative structure to the file.
  **CN L923:** 这一行为文件补充了实现细节或声明式结构。
- **EN L924:** Blank line used to separate nearby declarations and improve readability.
  **CN L924:** 该空行用于分隔相邻声明并提升可读性。

### Lines 925-936 / 第 925-936 行

```tablegen
 925:     Example:
 926: 
 927:     ```mlir
 928:     // Scalar log operation.
 929:     %y = math.log %x : f64
 930:     ```
 931:   }];
 932:   let hasFolder = 1;
 933: }
 934: 
 935: //===----------------------------------------------------------------------===//
 936: // Log10Op
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L925:** This line contributes implementation detail or declarative structure to the file.
  **CN L925:** 这一行为文件补充了实现细节或声明式结构。
- **EN L926:** Blank line used to separate nearby declarations and improve readability.
  **CN L926:** 该空行用于分隔相邻声明并提升可读性。
- **EN L927:** This line contributes implementation detail or declarative structure to the file.
  **CN L927:** 这一行为文件补充了实现细节或声明式结构。
- **EN L928:** This comment states: “Scalar log operation.”, documenting the intent of the surrounding code.
  **CN L928:** 该注释写道：“Scalar log operation.”，用于说明周围代码的意图。
- **EN L929:** This line contributes implementation detail or declarative structure to the file.
  **CN L929:** 这一行为文件补充了实现细节或声明式结构。
- **EN L930:** This line contributes implementation detail or declarative structure to the file.
  **CN L930:** 这一行为文件补充了实现细节或声明式结构。
- **EN L931:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L931:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L932:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L932:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L933:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L933:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L934:** Blank line used to separate nearby declarations and improve readability.
  **CN L934:** 该空行用于分隔相邻声明并提升可读性。
- **EN L935:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L935:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L936:** This comment states: “Log10Op”, documenting the intent of the surrounding code.
  **CN L936:** 该注释写道：“Log10Op”，用于说明周围代码的意图。

### Lines 937-948 / 第 937-948 行

```tablegen
 937: //===----------------------------------------------------------------------===//
 938: 
 939: def Math_Log10Op : Math_FloatUnaryOp<"log10"> {
 940:   let summary = "base-10 logarithm of the specified value";
 941: 
 942:   let description = [{
 943:     Computes the base-10 logarithm of the given value. It takes one operand of
 944:     floating point type (i.e., scalar, tensor or vector) and returns one result of
 945:     the same type.
 946: 
 947:     Example:
 948: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L937:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L937:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L938:** Blank line used to separate nearby declarations and improve readability.
  **CN L938:** 该空行用于分隔相邻声明并提升可读性。
- **EN L939:** This TableGen `def` record introduces `Math_Log10Op`, which later participates in generated MLIR code.
  **CN L939:** 该 TableGen `def` 记录引入了 `Math_Log10Op`，后续会参与生成的 MLIR 代码。
- **EN L940:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L940:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L941:** Blank line used to separate nearby declarations and improve readability.
  **CN L941:** 该空行用于分隔相邻声明并提升可读性。
- **EN L942:** This line contributes implementation detail or declarative structure to the file.
  **CN L942:** 这一行为文件补充了实现细节或声明式结构。
- **EN L943:** This line contributes implementation detail or declarative structure to the file.
  **CN L943:** 这一行为文件补充了实现细节或声明式结构。
- **EN L944:** This line contributes to the declaration or call of `type`.
  **CN L944:** 这一行为 `type` 的声明或调用提供内容。
- **EN L945:** This line contributes implementation detail or declarative structure to the file.
  **CN L945:** 这一行为文件补充了实现细节或声明式结构。
- **EN L946:** Blank line used to separate nearby declarations and improve readability.
  **CN L946:** 该空行用于分隔相邻声明并提升可读性。
- **EN L947:** This line contributes implementation detail or declarative structure to the file.
  **CN L947:** 这一行为文件补充了实现细节或声明式结构。
- **EN L948:** Blank line used to separate nearby declarations and improve readability.
  **CN L948:** 该空行用于分隔相邻声明并提升可读性。

### Lines 949-960 / 第 949-960 行

```tablegen
 949:     ```mlir
 950:     // Scalar log10 operation.
 951:     %y = math.log10 %x : f64
 952:     ```
 953:   }];
 954:   let hasFolder = 1;
 955: }
 956: 
 957: //===----------------------------------------------------------------------===//
 958: // Log1pOp
 959: //===----------------------------------------------------------------------===//
 960: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L949:** This line contributes implementation detail or declarative structure to the file.
  **CN L949:** 这一行为文件补充了实现细节或声明式结构。
- **EN L950:** This comment states: “Scalar log10 operation.”, documenting the intent of the surrounding code.
  **CN L950:** 该注释写道：“Scalar log10 operation.”，用于说明周围代码的意图。
- **EN L951:** This line contributes implementation detail or declarative structure to the file.
  **CN L951:** 这一行为文件补充了实现细节或声明式结构。
- **EN L952:** This line contributes implementation detail or declarative structure to the file.
  **CN L952:** 这一行为文件补充了实现细节或声明式结构。
- **EN L953:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L953:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L954:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L954:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L955:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L955:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L956:** Blank line used to separate nearby declarations and improve readability.
  **CN L956:** 该空行用于分隔相邻声明并提升可读性。
- **EN L957:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L957:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L958:** This comment states: “Log1pOp”, documenting the intent of the surrounding code.
  **CN L958:** 该注释写道：“Log1pOp”，用于说明周围代码的意图。
- **EN L959:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L959:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L960:** Blank line used to separate nearby declarations and improve readability.
  **CN L960:** 该空行用于分隔相邻声明并提升可读性。

### Lines 961-972 / 第 961-972 行

```tablegen
 961: def Math_Log1pOp : Math_FloatUnaryOp<"log1p"> {
 962:   let summary = "Computes the natural logarithm of one plus the given value";
 963: 
 964:   let description = [{
 965:     Computes the base-e logarithm of one plus the given value. It takes one
 966:     operand of floating point type (i.e., scalar, tensor or vector) and returns one
 967:     result of the same type.
 968: 
 969:     log1p(x) := log(1 + x)
 970: 
 971:     Example:
 972: 
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L961:** This TableGen `def` record introduces `Math_Log1pOp`, which later participates in generated MLIR code.
  **CN L961:** 该 TableGen `def` 记录引入了 `Math_Log1pOp`，后续会参与生成的 MLIR 代码。
- **EN L962:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L962:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L963:** Blank line used to separate nearby declarations and improve readability.
  **CN L963:** 该空行用于分隔相邻声明并提升可读性。
- **EN L964:** This line contributes implementation detail or declarative structure to the file.
  **CN L964:** 这一行为文件补充了实现细节或声明式结构。
- **EN L965:** This line contributes implementation detail or declarative structure to the file.
  **CN L965:** 这一行为文件补充了实现细节或声明式结构。
- **EN L966:** This line contributes to the declaration or call of `type`.
  **CN L966:** 这一行为 `type` 的声明或调用提供内容。
- **EN L967:** This line contributes implementation detail or declarative structure to the file.
  **CN L967:** 这一行为文件补充了实现细节或声明式结构。
- **EN L968:** Blank line used to separate nearby declarations and improve readability.
  **CN L968:** 该空行用于分隔相邻声明并提升可读性。
- **EN L969:** This line contributes to the declaration or call of `log1p`.
  **CN L969:** 这一行为 `log1p` 的声明或调用提供内容。
- **EN L970:** Blank line used to separate nearby declarations and improve readability.
  **CN L970:** 该空行用于分隔相邻声明并提升可读性。
- **EN L971:** This line contributes implementation detail or declarative structure to the file.
  **CN L971:** 这一行为文件补充了实现细节或声明式结构。
- **EN L972:** Blank line used to separate nearby declarations and improve readability.
  **CN L972:** 该空行用于分隔相邻声明并提升可读性。

### Lines 973-984 / 第 973-984 行

```tablegen
 973:     ```mlir
 974:     // Scalar log1p operation.
 975:     %y = math.log1p %x : f64
 976:     ```
 977:   }];
 978:   let hasFolder = 1;
 979: }
 980: 
 981: //===----------------------------------------------------------------------===//
 982: // Log2Op
 983: //===----------------------------------------------------------------------===//
 984: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L973:** This line contributes implementation detail or declarative structure to the file.
  **CN L973:** 这一行为文件补充了实现细节或声明式结构。
- **EN L974:** This comment states: “Scalar log1p operation.”, documenting the intent of the surrounding code.
  **CN L974:** 该注释写道：“Scalar log1p operation.”，用于说明周围代码的意图。
- **EN L975:** This line contributes implementation detail or declarative structure to the file.
  **CN L975:** 这一行为文件补充了实现细节或声明式结构。
- **EN L976:** This line contributes implementation detail or declarative structure to the file.
  **CN L976:** 这一行为文件补充了实现细节或声明式结构。
- **EN L977:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L977:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L978:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L978:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L979:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L979:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L980:** Blank line used to separate nearby declarations and improve readability.
  **CN L980:** 该空行用于分隔相邻声明并提升可读性。
- **EN L981:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L981:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L982:** This comment states: “Log2Op”, documenting the intent of the surrounding code.
  **CN L982:** 该注释写道：“Log2Op”，用于说明周围代码的意图。
- **EN L983:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L983:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L984:** Blank line used to separate nearby declarations and improve readability.
  **CN L984:** 该空行用于分隔相邻声明并提升可读性。

### Lines 985-996 / 第 985-996 行

```tablegen
 985: def Math_Log2Op : Math_FloatUnaryOp<"log2"> {
 986:   let summary = "base-2 logarithm of the specified value";
 987: 
 988:   let description = [{
 989:     Computes the base-2 logarithm of the given value. It takes one operand of
 990:     floating point type (i.e., scalar, tensor or vector) and returns one result of
 991:     the same type.
 992: 
 993:     Example:
 994: 
 995:     ```mlir
 996:     // Scalar log2 operation.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L985:** This TableGen `def` record introduces `Math_Log2Op`, which later participates in generated MLIR code.
  **CN L985:** 该 TableGen `def` 记录引入了 `Math_Log2Op`，后续会参与生成的 MLIR 代码。
- **EN L986:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L986:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L987:** Blank line used to separate nearby declarations and improve readability.
  **CN L987:** 该空行用于分隔相邻声明并提升可读性。
- **EN L988:** This line contributes implementation detail or declarative structure to the file.
  **CN L988:** 这一行为文件补充了实现细节或声明式结构。
- **EN L989:** This line contributes implementation detail or declarative structure to the file.
  **CN L989:** 这一行为文件补充了实现细节或声明式结构。
- **EN L990:** This line contributes to the declaration or call of `type`.
  **CN L990:** 这一行为 `type` 的声明或调用提供内容。
- **EN L991:** This line contributes implementation detail or declarative structure to the file.
  **CN L991:** 这一行为文件补充了实现细节或声明式结构。
- **EN L992:** Blank line used to separate nearby declarations and improve readability.
  **CN L992:** 该空行用于分隔相邻声明并提升可读性。
- **EN L993:** This line contributes implementation detail or declarative structure to the file.
  **CN L993:** 这一行为文件补充了实现细节或声明式结构。
- **EN L994:** Blank line used to separate nearby declarations and improve readability.
  **CN L994:** 该空行用于分隔相邻声明并提升可读性。
- **EN L995:** This line contributes implementation detail or declarative structure to the file.
  **CN L995:** 这一行为文件补充了实现细节或声明式结构。
- **EN L996:** This comment states: “Scalar log2 operation.”, documenting the intent of the surrounding code.
  **CN L996:** 该注释写道：“Scalar log2 operation.”，用于说明周围代码的意图。

### Lines 997-1008 / 第 997-1008 行

```tablegen
 997:     %y = math.log2 %x : f64
 998:     ```
 999:   }];
1000:   let hasFolder = 1;
1001: }
1002: 
1003: //===----------------------------------------------------------------------===//
1004: // PowFOp
1005: //===----------------------------------------------------------------------===//
1006: 
1007: def Math_PowFOp : Math_FloatBinaryOp<"powf"> {
1008:   let summary = "floating point raised to the power of operation";
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L997:** This line contributes implementation detail or declarative structure to the file.
  **CN L997:** 这一行为文件补充了实现细节或声明式结构。
- **EN L998:** This line contributes implementation detail or declarative structure to the file.
  **CN L998:** 这一行为文件补充了实现细节或声明式结构。
- **EN L999:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L999:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1000:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1000:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1001:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1001:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1002:** Blank line used to separate nearby declarations and improve readability.
  **CN L1002:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1003:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1003:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1004:** This comment states: “PowFOp”, documenting the intent of the surrounding code.
  **CN L1004:** 该注释写道：“PowFOp”，用于说明周围代码的意图。
- **EN L1005:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1005:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1006:** Blank line used to separate nearby declarations and improve readability.
  **CN L1006:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1007:** This TableGen `def` record introduces `Math_PowFOp`, which later participates in generated MLIR code.
  **CN L1007:** 该 TableGen `def` 记录引入了 `Math_PowFOp`，后续会参与生成的 MLIR 代码。
- **EN L1008:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1008:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1009-1020 / 第 1009-1020 行

```tablegen
1009:   let description = [{
1010:     The `powf` operation takes two operands of floating point type (i.e.,
1011:     scalar, tensor or vector) and returns one result of the same type. Operands
1012:     must have the same type.
1013: 
1014:     Example:
1015: 
1016:     ```mlir
1017:     // Scalar exponentiation.
1018:     %a = math.powf %b, %c : f64
1019:     ```
1020:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1009:** This line contributes implementation detail or declarative structure to the file.
  **CN L1009:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1010:** This line contributes to the declaration or call of `type`.
  **CN L1010:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1011:** This line contributes implementation detail or declarative structure to the file.
  **CN L1011:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1012:** This line contributes implementation detail or declarative structure to the file.
  **CN L1012:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1013:** Blank line used to separate nearby declarations and improve readability.
  **CN L1013:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1014:** This line contributes implementation detail or declarative structure to the file.
  **CN L1014:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1015:** Blank line used to separate nearby declarations and improve readability.
  **CN L1015:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1016:** This line contributes implementation detail or declarative structure to the file.
  **CN L1016:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1017:** This comment states: “Scalar exponentiation.”, documenting the intent of the surrounding code.
  **CN L1017:** 该注释写道：“Scalar exponentiation.”，用于说明周围代码的意图。
- **EN L1018:** This line contributes implementation detail or declarative structure to the file.
  **CN L1018:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1019:** This line contributes implementation detail or declarative structure to the file.
  **CN L1019:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1020:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1020:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1021-1032 / 第 1021-1032 行

```tablegen
1021:   let hasFolder = 1;
1022: }
1023: 
1024: //===----------------------------------------------------------------------===//
1025: // RsqrtOp
1026: //===----------------------------------------------------------------------===//
1027: 
1028: def Math_RsqrtOp : Math_FloatUnaryOp<"rsqrt"> {
1029:   let summary = "reciprocal of sqrt (1 / sqrt of the specified value)";
1030:   let description = [{
1031:     The `rsqrt` operation computes the reciprocal of the square root. It takes
1032:     one operand of floating point type (i.e., scalar, tensor or vector) and returns
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1021:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1021:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1022:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1022:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1023:** Blank line used to separate nearby declarations and improve readability.
  **CN L1023:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1024:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1024:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1025:** This comment states: “RsqrtOp”, documenting the intent of the surrounding code.
  **CN L1025:** 该注释写道：“RsqrtOp”，用于说明周围代码的意图。
- **EN L1026:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1026:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1027:** Blank line used to separate nearby declarations and improve readability.
  **CN L1027:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1028:** This TableGen `def` record introduces `Math_RsqrtOp`, which later participates in generated MLIR code.
  **CN L1028:** 该 TableGen `def` 记录引入了 `Math_RsqrtOp`，后续会参与生成的 MLIR 代码。
- **EN L1029:** This line contributes to the declaration or call of `sqrt`.
  **CN L1029:** 这一行为 `sqrt` 的声明或调用提供内容。
- **EN L1030:** This line contributes implementation detail or declarative structure to the file.
  **CN L1030:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1031:** This line contributes implementation detail or declarative structure to the file.
  **CN L1031:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1032:** This line contributes to the declaration or call of `type`.
  **CN L1032:** 这一行为 `type` 的声明或调用提供内容。

### Lines 1033-1044 / 第 1033-1044 行

```tablegen
1033:     one result of the same type. It has no standard attributes.
1034: 
1035:     Example:
1036: 
1037:     ```mlir
1038:     // Scalar reciprocal square root value.
1039:     %a = math.rsqrt %b : f64
1040:     ```
1041:   }];
1042:   let hasFolder = 1;
1043: }
1044: 
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1033:** This line contributes implementation detail or declarative structure to the file.
  **CN L1033:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1034:** Blank line used to separate nearby declarations and improve readability.
  **CN L1034:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1035:** This line contributes implementation detail or declarative structure to the file.
  **CN L1035:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1036:** Blank line used to separate nearby declarations and improve readability.
  **CN L1036:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1037:** This line contributes implementation detail or declarative structure to the file.
  **CN L1037:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1038:** This comment states: “Scalar reciprocal square root value.”, documenting the intent of the surrounding code.
  **CN L1038:** 该注释写道：“Scalar reciprocal square root value.”，用于说明周围代码的意图。
- **EN L1039:** This line contributes implementation detail or declarative structure to the file.
  **CN L1039:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1040:** This line contributes implementation detail or declarative structure to the file.
  **CN L1040:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1041:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1041:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1042:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1042:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1043:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1043:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1044:** Blank line used to separate nearby declarations and improve readability.
  **CN L1044:** 该空行用于分隔相邻声明并提升可读性。

### Lines 1045-1056 / 第 1045-1056 行

```tablegen
1045: //===----------------------------------------------------------------------===//
1046: // SqrtOp
1047: //===----------------------------------------------------------------------===//
1048: 
1049: def Math_SqrtOp : Math_FloatUnaryOp<"sqrt"> {
1050:   let summary = "sqrt of the specified value";
1051:   let description = [{
1052:     The `sqrt` operation computes the square root. It takes one operand of
1053:     floating point type (i.e., scalar, tensor or vector) and returns one result of
1054:     the same type. It has no standard attributes.
1055: 
1056:     Example:
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1045:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1045:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1046:** This comment states: “SqrtOp”, documenting the intent of the surrounding code.
  **CN L1046:** 该注释写道：“SqrtOp”，用于说明周围代码的意图。
- **EN L1047:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1047:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1048:** Blank line used to separate nearby declarations and improve readability.
  **CN L1048:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1049:** This TableGen `def` record introduces `Math_SqrtOp`, which later participates in generated MLIR code.
  **CN L1049:** 该 TableGen `def` 记录引入了 `Math_SqrtOp`，后续会参与生成的 MLIR 代码。
- **EN L1050:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1050:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1051:** This line contributes implementation detail or declarative structure to the file.
  **CN L1051:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1052:** This line contributes implementation detail or declarative structure to the file.
  **CN L1052:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1053:** This line contributes to the declaration or call of `type`.
  **CN L1053:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1054:** This line contributes implementation detail or declarative structure to the file.
  **CN L1054:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1055:** Blank line used to separate nearby declarations and improve readability.
  **CN L1055:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1056:** This line contributes implementation detail or declarative structure to the file.
  **CN L1056:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1057-1068 / 第 1057-1068 行

```tablegen
1057: 
1058:     ```mlir
1059:     // Scalar square root value.
1060:     %a = math.sqrt %b : f64
1061:     ```
1062:   }];
1063:   let hasFolder = 1;
1064: }
1065: 
1066: //===----------------------------------------------------------------------===//
1067: // TanOp
1068: //===----------------------------------------------------------------------===//
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1057:** Blank line used to separate nearby declarations and improve readability.
  **CN L1057:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1058:** This line contributes implementation detail or declarative structure to the file.
  **CN L1058:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1059:** This comment states: “Scalar square root value.”, documenting the intent of the surrounding code.
  **CN L1059:** 该注释写道：“Scalar square root value.”，用于说明周围代码的意图。
- **EN L1060:** This line contributes implementation detail or declarative structure to the file.
  **CN L1060:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1061:** This line contributes implementation detail or declarative structure to the file.
  **CN L1061:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1062:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1062:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1063:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1063:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1064:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1064:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1065:** Blank line used to separate nearby declarations and improve readability.
  **CN L1065:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1066:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1066:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1067:** This comment states: “TanOp”, documenting the intent of the surrounding code.
  **CN L1067:** 该注释写道：“TanOp”，用于说明周围代码的意图。
- **EN L1068:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1068:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。

### Lines 1069-1080 / 第 1069-1080 行

```tablegen
1069: 
1070: def Math_TanOp : Math_FloatUnaryOp<"tan"> {
1071:   let summary = "tangent of the specified value";
1072:   let description = [{
1073:     The `tan` operation computes the tangent. It takes one operand
1074:     of floating point type (i.e., scalar, tensor or vector) and returns one
1075:     result of the same type. It has no standard attributes.
1076: 
1077:     Example:
1078: 
1079:     ```mlir
1080:     // Scalar tangent value.
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1069:** Blank line used to separate nearby declarations and improve readability.
  **CN L1069:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1070:** This TableGen `def` record introduces `Math_TanOp`, which later participates in generated MLIR code.
  **CN L1070:** 该 TableGen `def` 记录引入了 `Math_TanOp`，后续会参与生成的 MLIR 代码。
- **EN L1071:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1071:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1072:** This line contributes implementation detail or declarative structure to the file.
  **CN L1072:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1073:** This line contributes implementation detail or declarative structure to the file.
  **CN L1073:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1074:** This line contributes to the declaration or call of `type`.
  **CN L1074:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1075:** This line contributes implementation detail or declarative structure to the file.
  **CN L1075:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1076:** Blank line used to separate nearby declarations and improve readability.
  **CN L1076:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1077:** This line contributes implementation detail or declarative structure to the file.
  **CN L1077:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1078:** Blank line used to separate nearby declarations and improve readability.
  **CN L1078:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1079:** This line contributes implementation detail or declarative structure to the file.
  **CN L1079:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1080:** This comment states: “Scalar tangent value.”, documenting the intent of the surrounding code.
  **CN L1080:** 该注释写道：“Scalar tangent value.”，用于说明周围代码的意图。

### Lines 1081-1092 / 第 1081-1092 行

```tablegen
1081:     %a = math.tan %b : f64
1082:     ```
1083:   }];
1084:   let hasFolder = 1;
1085: }
1086: 
1087: //===----------------------------------------------------------------------===//
1088: // TanhOp
1089: //===----------------------------------------------------------------------===//
1090: 
1091: def Math_TanhOp : Math_FloatUnaryOp<"tanh"> {
1092:   let summary = "hyperbolic tangent of the specified value";
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1081:** This line contributes implementation detail or declarative structure to the file.
  **CN L1081:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1082:** This line contributes implementation detail or declarative structure to the file.
  **CN L1082:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1083:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1083:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1084:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1084:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1085:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1085:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1086:** Blank line used to separate nearby declarations and improve readability.
  **CN L1086:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1087:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1087:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1088:** This comment states: “TanhOp”, documenting the intent of the surrounding code.
  **CN L1088:** 该注释写道：“TanhOp”，用于说明周围代码的意图。
- **EN L1089:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1089:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1090:** Blank line used to separate nearby declarations and improve readability.
  **CN L1090:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1091:** This TableGen `def` record introduces `Math_TanhOp`, which later participates in generated MLIR code.
  **CN L1091:** 该 TableGen `def` 记录引入了 `Math_TanhOp`，后续会参与生成的 MLIR 代码。
- **EN L1092:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1092:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1093-1104 / 第 1093-1104 行

```tablegen
1093:   let description = [{
1094:     The `tanh` operation computes the hyperbolic tangent. It takes one operand
1095:     of floating point type (i.e., scalar, tensor or vector) and returns one
1096:     result of the same type. It has no standard attributes.
1097: 
1098:     Example:
1099: 
1100:     ```mlir
1101:     // Scalar hyperbolic tangent value.
1102:     %a = math.tanh %b : f64
1103:     ```
1104:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1093:** This line contributes implementation detail or declarative structure to the file.
  **CN L1093:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1094:** This line contributes implementation detail or declarative structure to the file.
  **CN L1094:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1095:** This line contributes to the declaration or call of `type`.
  **CN L1095:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1096:** This line contributes implementation detail or declarative structure to the file.
  **CN L1096:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1097:** Blank line used to separate nearby declarations and improve readability.
  **CN L1097:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1098:** This line contributes implementation detail or declarative structure to the file.
  **CN L1098:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1099:** Blank line used to separate nearby declarations and improve readability.
  **CN L1099:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1100:** This line contributes implementation detail or declarative structure to the file.
  **CN L1100:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1101:** This comment states: “Scalar hyperbolic tangent value.”, documenting the intent of the surrounding code.
  **CN L1101:** 该注释写道：“Scalar hyperbolic tangent value.”，用于说明周围代码的意图。
- **EN L1102:** This line contributes implementation detail or declarative structure to the file.
  **CN L1102:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1103:** This line contributes implementation detail or declarative structure to the file.
  **CN L1103:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1104:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1104:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1105-1116 / 第 1105-1116 行

```tablegen
1105:   let hasFolder = 1;
1106: }
1107: 
1108: //===----------------------------------------------------------------------===//
1109: // RoundEvenOp
1110: //===----------------------------------------------------------------------===//
1111: 
1112: def Math_RoundEvenOp : Math_FloatUnaryOp<"roundeven"> {
1113:   let summary = "round of the specified value with halfway cases to even";
1114:   let description = [{
1115:     The `roundeven` operation returns the operand rounded to the nearest integer
1116:     value in floating-point format. It takes one operand of floating point type
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1105:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1105:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1106:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1106:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1107:** Blank line used to separate nearby declarations and improve readability.
  **CN L1107:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1108:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1108:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1109:** This comment states: “RoundEvenOp”, documenting the intent of the surrounding code.
  **CN L1109:** 该注释写道：“RoundEvenOp”，用于说明周围代码的意图。
- **EN L1110:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1110:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1111:** Blank line used to separate nearby declarations and improve readability.
  **CN L1111:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1112:** This TableGen `def` record introduces `Math_RoundEvenOp`, which later participates in generated MLIR code.
  **CN L1112:** 该 TableGen `def` 记录引入了 `Math_RoundEvenOp`，后续会参与生成的 MLIR 代码。
- **EN L1113:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1113:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1114:** This line contributes implementation detail or declarative structure to the file.
  **CN L1114:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1115:** This line contributes implementation detail or declarative structure to the file.
  **CN L1115:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1116:** This line contributes implementation detail or declarative structure to the file.
  **CN L1116:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1117-1128 / 第 1117-1128 行

```tablegen
1117:     (i.e., scalar, tensor or vector) and produces one result of the same type.  The
1118:     operation rounds the argument to the nearest integer value in floating-point
1119:     format, rounding halfway cases to even, regardless of the current
1120:     rounding direction.
1121: 
1122:     Example:
1123: 
1124:     ```mlir
1125:     // Scalar round operation.
1126:     %a = math.roundeven %b : f64
1127:     ```
1128:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1117:** This line contributes implementation detail or declarative structure to the file.
  **CN L1117:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1118:** This line contributes implementation detail or declarative structure to the file.
  **CN L1118:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1119:** This line contributes implementation detail or declarative structure to the file.
  **CN L1119:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1120:** This line contributes implementation detail or declarative structure to the file.
  **CN L1120:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1121:** Blank line used to separate nearby declarations and improve readability.
  **CN L1121:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1122:** This line contributes implementation detail or declarative structure to the file.
  **CN L1122:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1123:** Blank line used to separate nearby declarations and improve readability.
  **CN L1123:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1124:** This line contributes implementation detail or declarative structure to the file.
  **CN L1124:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1125:** This comment states: “Scalar round operation.”, documenting the intent of the surrounding code.
  **CN L1125:** 该注释写道：“Scalar round operation.”，用于说明周围代码的意图。
- **EN L1126:** This line contributes implementation detail or declarative structure to the file.
  **CN L1126:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1127:** This line contributes implementation detail or declarative structure to the file.
  **CN L1127:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1128:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1128:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1129-1140 / 第 1129-1140 行

```tablegen
1129:   let hasFolder = 1;
1130: }
1131: 
1132: //===----------------------------------------------------------------------===//
1133: // RoundOp
1134: //===----------------------------------------------------------------------===//
1135: 
1136: def Math_RoundOp : Math_FloatUnaryOp<"round"> {
1137:   let summary = "round of the specified value";
1138:   let description = [{
1139:     The `round` operation returns the operand rounded to the nearest integer
1140:     value in floating-point format. It takes one operand of floating point type
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1129:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1129:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1130:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1130:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1131:** Blank line used to separate nearby declarations and improve readability.
  **CN L1131:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1132:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1132:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1133:** This comment states: “RoundOp”, documenting the intent of the surrounding code.
  **CN L1133:** 该注释写道：“RoundOp”，用于说明周围代码的意图。
- **EN L1134:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1134:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1135:** Blank line used to separate nearby declarations and improve readability.
  **CN L1135:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1136:** This TableGen `def` record introduces `Math_RoundOp`, which later participates in generated MLIR code.
  **CN L1136:** 该 TableGen `def` 记录引入了 `Math_RoundOp`，后续会参与生成的 MLIR 代码。
- **EN L1137:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1137:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1138:** This line contributes implementation detail or declarative structure to the file.
  **CN L1138:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1139:** This line contributes implementation detail or declarative structure to the file.
  **CN L1139:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1140:** This line contributes implementation detail or declarative structure to the file.
  **CN L1140:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1141-1152 / 第 1141-1152 行

```tablegen
1141:     (i.e., scalar, tensor or vector) and produces one result of the same type.  The
1142:     operation rounds the argument to the nearest integer value in floating-point
1143:     format, rounding halfway cases away from zero, regardless of the current
1144:     rounding direction.
1145: 
1146:     Example:
1147: 
1148:     ```mlir
1149:     // Scalar round operation.
1150:     %a = math.round %b : f64
1151:     ```
1152:   }];
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1141:** This line contributes implementation detail or declarative structure to the file.
  **CN L1141:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1142:** This line contributes implementation detail or declarative structure to the file.
  **CN L1142:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1143:** This line contributes implementation detail or declarative structure to the file.
  **CN L1143:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1144:** This line contributes implementation detail or declarative structure to the file.
  **CN L1144:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1145:** Blank line used to separate nearby declarations and improve readability.
  **CN L1145:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1146:** This line contributes implementation detail or declarative structure to the file.
  **CN L1146:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1147:** Blank line used to separate nearby declarations and improve readability.
  **CN L1147:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1148:** This line contributes implementation detail or declarative structure to the file.
  **CN L1148:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1149:** This comment states: “Scalar round operation.”, documenting the intent of the surrounding code.
  **CN L1149:** 该注释写道：“Scalar round operation.”，用于说明周围代码的意图。
- **EN L1150:** This line contributes implementation detail or declarative structure to the file.
  **CN L1150:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1151:** This line contributes implementation detail or declarative structure to the file.
  **CN L1151:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1152:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1152:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1153-1164 / 第 1153-1164 行

```tablegen
1153:   let hasFolder = 1;
1154: }
1155: 
1156: //===----------------------------------------------------------------------===//
1157: // TruncOp
1158: //===----------------------------------------------------------------------===//
1159: 
1160: def Math_TruncOp : Math_FloatUnaryOp<"trunc"> {
1161:   let summary = "trunc of the specified value";
1162:   let description = [{
1163:     The `trunc` operation returns the operand rounded to the nearest integer
1164:     value in floating-point format. It takes one operand of floating point type
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1153:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1153:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1154:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1154:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1155:** Blank line used to separate nearby declarations and improve readability.
  **CN L1155:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1156:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1156:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1157:** This comment states: “TruncOp”, documenting the intent of the surrounding code.
  **CN L1157:** 该注释写道：“TruncOp”，用于说明周围代码的意图。
- **EN L1158:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1158:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1159:** Blank line used to separate nearby declarations and improve readability.
  **CN L1159:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1160:** This TableGen `def` record introduces `Math_TruncOp`, which later participates in generated MLIR code.
  **CN L1160:** 该 TableGen `def` 记录引入了 `Math_TruncOp`，后续会参与生成的 MLIR 代码。
- **EN L1161:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1161:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1162:** This line contributes implementation detail or declarative structure to the file.
  **CN L1162:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1163:** This line contributes implementation detail or declarative structure to the file.
  **CN L1163:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1164:** This line contributes implementation detail or declarative structure to the file.
  **CN L1164:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1165-1176 / 第 1165-1176 行

```tablegen
1165:     (i.e., scalar, tensor or vector) and produces one result of the same type.
1166:     The operation always rounds to the nearest integer not larger in magnitude
1167:     than the operand, regardless of the current rounding direction.
1168: 
1169:     Example:
1170: 
1171:     ```mlir
1172:     // Scalar trunc operation.
1173:     %a = math.trunc %b : f64
1174:     ```
1175:   }];
1176:   let hasFolder = 1;
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1165:** This line contributes implementation detail or declarative structure to the file.
  **CN L1165:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1166:** This line contributes implementation detail or declarative structure to the file.
  **CN L1166:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1167:** This line contributes implementation detail or declarative structure to the file.
  **CN L1167:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1168:** Blank line used to separate nearby declarations and improve readability.
  **CN L1168:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1169:** This line contributes implementation detail or declarative structure to the file.
  **CN L1169:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1170:** Blank line used to separate nearby declarations and improve readability.
  **CN L1170:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1171:** This line contributes implementation detail or declarative structure to the file.
  **CN L1171:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1172:** This comment states: “Scalar trunc operation.”, documenting the intent of the surrounding code.
  **CN L1172:** 该注释写道：“Scalar trunc operation.”，用于说明周围代码的意图。
- **EN L1173:** This line contributes implementation detail or declarative structure to the file.
  **CN L1173:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1174:** This line contributes implementation detail or declarative structure to the file.
  **CN L1174:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1175:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1175:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1176:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1176:** 该语句或声明扩展了周围的 API 表面或存储状态。

### Lines 1177-1188 / 第 1177-1188 行

```tablegen
1177: }
1178: 
1179: //===----------------------------------------------------------------------===//
1180: // FPowIOp
1181: //===----------------------------------------------------------------------===//
1182: 
1183: def Math_FPowIOp : Math_Op<"fpowi",
1184:     [SameOperandsAndResultShape, AllTypesMatch<["lhs", "result"]>,
1185:      DeclareOpInterfaceMethods<ArithFastMathInterface>]> {
1186:   let summary = "floating point raised to the signed integer power";
1187:   let description = [{
1188:     The `fpowi` operation takes a `base` operand of floating point type
```

- **EN:** Generation — This block encodes declarative metadata that will be expanded by TableGen.
  **CN:** 代码生成——这一段编码了会被 TableGen 展开的声明式元数据。
- **EN L1177:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1177:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1178:** Blank line used to separate nearby declarations and improve readability.
  **CN L1178:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1179:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1179:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1180:** This comment states: “FPowIOp”, documenting the intent of the surrounding code.
  **CN L1180:** 该注释写道：“FPowIOp”，用于说明周围代码的意图。
- **EN L1181:** This comment states: “===----------------------------------------------------------------------===”, documenting the intent of the surrounding code.
  **CN L1181:** 该注释写道：“===----------------------------------------------------------------------===”，用于说明周围代码的意图。
- **EN L1182:** Blank line used to separate nearby declarations and improve readability.
  **CN L1182:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1183:** This TableGen `def` record introduces `Math_FPowIOp`, which later participates in generated MLIR code.
  **CN L1183:** 该 TableGen `def` 记录引入了 `Math_FPowIOp`，后续会参与生成的 MLIR 代码。
- **EN L1184:** This line contributes implementation detail or declarative structure to the file.
  **CN L1184:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1185:** This line contributes implementation detail or declarative structure to the file.
  **CN L1185:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1186:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1186:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1187:** This line contributes implementation detail or declarative structure to the file.
  **CN L1187:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1188:** This line contributes implementation detail or declarative structure to the file.
  **CN L1188:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1189-1200 / 第 1189-1200 行

```tablegen
1189:     (i.e. scalar, tensor or vector) and a `power` operand of integer type
1190:     (also scalar, tensor or vector) and returns one result of the same type
1191:     as `base`. The result is `base` raised to the power of `power`.
1192:     The operation is elementwise for non-scalars, e.g.:
1193: 
1194:     ```mlir
1195:     %v = math.fpowi %base, %power : vector<2xf32>, vector<2xi32>
1196:     ```
1197: 
1198:     The result is a vector of:
1199: 
1200:     ```
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1189:** This line contributes implementation detail or declarative structure to the file.
  **CN L1189:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1190:** This line contributes implementation detail or declarative structure to the file.
  **CN L1190:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1191:** This line contributes implementation detail or declarative structure to the file.
  **CN L1191:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1192:** This line contributes implementation detail or declarative structure to the file.
  **CN L1192:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1193:** Blank line used to separate nearby declarations and improve readability.
  **CN L1193:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1194:** This line contributes implementation detail or declarative structure to the file.
  **CN L1194:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1195:** This line contributes implementation detail or declarative structure to the file.
  **CN L1195:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1196:** This line contributes implementation detail or declarative structure to the file.
  **CN L1196:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1197:** Blank line used to separate nearby declarations and improve readability.
  **CN L1197:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1198:** This line contributes implementation detail or declarative structure to the file.
  **CN L1198:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1199:** Blank line used to separate nearby declarations and improve readability.
  **CN L1199:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1200:** This line contributes implementation detail or declarative structure to the file.
  **CN L1200:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1201-1212 / 第 1201-1212 行

```tablegen
1201:     [<math.fpowi %base[0], %power[0]>, <math.fpowi %base[1], %power[1]>]
1202:     ```
1203: 
1204:     Example:
1205: 
1206:     ```mlir
1207:     // Scalar exponentiation.
1208:     %a = math.fpowi %base, %power : f64, i32
1209:     ```
1210:   }];
1211: 
1212:   let arguments = (ins FloatLike:$lhs, SignlessIntegerOrIndexLike:$rhs,
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1201:** This line contributes implementation detail or declarative structure to the file.
  **CN L1201:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1202:** This line contributes implementation detail or declarative structure to the file.
  **CN L1202:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1203:** Blank line used to separate nearby declarations and improve readability.
  **CN L1203:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1204:** This line contributes implementation detail or declarative structure to the file.
  **CN L1204:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1205:** Blank line used to separate nearby declarations and improve readability.
  **CN L1205:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1206:** This line contributes implementation detail or declarative structure to the file.
  **CN L1206:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1207:** This comment states: “Scalar exponentiation.”, documenting the intent of the surrounding code.
  **CN L1207:** 该注释写道：“Scalar exponentiation.”，用于说明周围代码的意图。
- **EN L1208:** This line contributes implementation detail or declarative structure to the file.
  **CN L1208:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1209:** This line contributes implementation detail or declarative structure to the file.
  **CN L1209:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1210:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1210:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1211:** Blank line used to separate nearby declarations and improve readability.
  **CN L1211:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1212:** This line contributes implementation detail or declarative structure to the file.
  **CN L1212:** 这一行为文件补充了实现细节或声明式结构。

### Lines 1213-1222 / 第 1213-1222 行

```tablegen
1213:       DefaultValuedAttr<Arith_FastMathAttr,
1214:                         "::mlir::arith::FastMathFlags::none">:$fastmath);
1215:   let results = (outs FloatLike:$result);
1216:   let assemblyFormat = [{ $lhs `,` $rhs (`fastmath` `` $fastmath^)?
1217:                           attr-dict `:` type($lhs) `,` type($rhs) }];
1218: 
1219:   let hasFolder = 1;
1220: }
1221: 
1222: #endif // MATH_OPS
```

- **EN:** Structure — This block provides supporting declarations, comments, or glue logic.
  **CN:** 结构——这一段提供支撑性声明、注释或胶水逻辑。
- **EN L1213:** This line contributes implementation detail or declarative structure to the file.
  **CN L1213:** 这一行为文件补充了实现细节或声明式结构。
- **EN L1214:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1214:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1215:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1215:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1216:** This line contributes to the declaration or call of `rhs`.
  **CN L1216:** 这一行为 `rhs` 的声明或调用提供内容。
- **EN L1217:** This line contributes to the declaration or call of `type`.
  **CN L1217:** 这一行为 `type` 的声明或调用提供内容。
- **EN L1218:** Blank line used to separate nearby declarations and improve readability.
  **CN L1218:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1219:** This statement or declaration extends the surrounding API surface or stored state.
  **CN L1219:** 该语句或声明扩展了周围的 API 表面或存储状态。
- **EN L1220:** This line closes the current scope, such as a namespace, class, or enum block.
  **CN L1220:** 这一行结束当前作用域，例如命名空间、类或枚举块。
- **EN L1221:** Blank line used to separate nearby declarations and improve readability.
  **CN L1221:** 该空行用于分隔相邻声明并提升可读性。
- **EN L1222:** This directive closes the conditional compilation region guarded by `MATH_OPS`.
  **CN L1222:** 该指令结束了由 `MATH_OPS` 保护的条件编译区域。

## Key Concepts / 关键概念

- **Math_Op**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **Math_IntegerUnaryOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **Math_FloatClassificationOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **Math_FloatUnaryOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **Math_IntegerBinaryOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **Math_FloatBinaryOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **Math_FloatTernaryOp**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。
- **Math_FloatTernaryOpWithRoundingMode**  
  EN: A declarative TableGen symbol that contributes to generated MLIR artifacts.  
  CN: 这是会参与生成 MLIR 产物的声明式 TableGen 符号。

## Dependencies / 依赖关系

- **`mlir/Dialect/Arith/IR/ArithBase.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Dialect/Arith/IR/ArithOpsInterfaces.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Dialect/Math/IR/MathBase.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Interfaces/InferTypeOpInterface.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Interfaces/VectorInterfaces.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。
- **`mlir/Interfaces/SideEffectInterfaces.td`**  
  EN: TableGen include reused during dialect/interface generation.  
  CN: 方言/接口生成过程中复用的 TableGen include。

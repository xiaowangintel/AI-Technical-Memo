# HexagonCallingConv.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonCallingConv.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon calling-convention rules using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```tablegen
     1: //===- HexagonCallingConv.td ----------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: // We cannot use the standard CCIfArgVarArg class here since in Hexagon there
    10: // exists a special case for a musl environment.  In a musl environment VarArgs
    11: // are treated like non VarArgs.  I.e., in a musl enviroment unnamed arguments
    12: // can also be passed in registers.  The CCIfArgVarArg class only checks each
    13: // individual argument, but not whether State.isVarArg() is true.  We also have
    14: // to check State.isVarArg() which is determined by the TreatAsVarArg argument.
    15: class CCIfStateVarArgAndArgVarArg<CCAction A>
    16:   : CCIf<"State.isVarArg() && ArgFlags.isVarArg()", A>;
    17: 
    18: def CC_HexagonStack: CallingConv<[
    19:   CCIfType<[i32,v2i16,v4i8],
    20:     CCAssignToStack<4,4>>,
    21:   CCIfType<[i64,v2i32,v4i16,v8i8],
    22:     CCAssignToStack<8,8>>
    23: ]>;
    24: 
    25: def CC_Hexagon_Legacy: CallingConv<[
```
- EN: It declares types such as CCIfStateVarArgAndArgVarArg, which carry the state or API of this component. It defines declarative TableGen records like CCIfStateVarArgAndArgVarArg, CC_HexagonStack, CC_Hexagon_Legacy; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCallingConv, showing how the code connects to sibling backend components.
- CN: 这里声明了 CCIfStateVarArgAndArgVarArg 等类型，用来承载该组件的状态或接口。 这里定义了 CCIfStateVarArgAndArgVarArg, CC_HexagonStack, CC_Hexagon_Legacy 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCallingConv，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```tablegen
    26:   CCIfType<[i1,i8,i16],
    27:     CCPromoteToType<i32>>,
    28:   CCIfType<[bf16],
    29:     CCBitConvertToType<i32>>,
    30:   CCIfType<[f32],
    31:     CCBitConvertToType<i32>>,
    32:   CCIfType<[f64],
    33:     CCBitConvertToType<i64>>,
    34: 
    35:   CCIfByVal<
    36:     CCPassByVal<8,8>>,
    37:   CCIfStateVarArgAndArgVarArg<
    38:     CCDelegateTo<CC_HexagonStack>>,
    39: 
    40:   // Pass split values in pairs, allocate odd register if necessary.
    41:   CCIfType<[i32],
    42:     CCIfSplit<
    43:       CCCustom<"CC_SkipOdd">>>,
    44: 
    45:   CCIfType<[i32,v2i16,v4i8],
    46:     CCAssignToReg<[R0,R1,R2,R3,R4,R5]>>,
    47:   // Make sure to allocate any skipped 32-bit register, so it does not get
    48:   // allocated to a subsequent 32-bit value.
    49:   CCIfType<[i64,v2i32,v4i16,v8i8],
    50:     CCCustom<"CC_SkipOdd">>,
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 51-75 / 第 51-75 行

```tablegen
    51:   CCIfType<[i64,v2i32,v4i16,v8i8],
    52:     CCAssignToReg<[D0,D1,D2]>>,
    53: 
    54:   CCDelegateTo<CC_HexagonStack>
    55: ]>;
    56: 
    57: def CC_Hexagon: CallingConv<[
    58:   CCIfType<[i1,i8,i16],
    59:     CCPromoteToType<i32>>,
    60:   CCIfType<[bf16],
    61:     CCBitConvertToType<i32>>,
    62:   CCIfType<[f32],
    63:     CCBitConvertToType<i32>>,
    64:   CCIfType<[f64],
    65:     CCBitConvertToType<i64>>,
    66: 
    67:   CCIfByVal<
    68:     CCPassByVal<8,1>>,
    69:   CCIfStateVarArgAndArgVarArg<
    70:     CCDelegateTo<CC_HexagonStack>>,
    71: 
    72:   // Pass split values in pairs, allocate odd register if necessary.
    73:   CCIfType<[i32],
    74:     CCIfSplit<
    75:       CCCustom<"CC_SkipOdd">>>,
```
- EN: It defines declarative TableGen records like CC_Hexagon; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里定义了 CC_Hexagon 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 76-100 / 第 76-100 行

```tablegen
    76:   CCIfType<[v2i1],  CCPromoteToType<v2i32>>,
    77:   CCIfType<[v4i1],  CCPromoteToType<v4i16>>,
    78:   CCIfType<[v8i1],  CCPromoteToType<v8i8>>,
    79: 
    80:   CCIfType<[i32,v2i16,v4i8],
    81:     CCAssignToReg<[R0,R1,R2,R3,R4,R5]>>,
    82:   // Make sure to allocate any skipped 32-bit register, so it does not get
    83:   // allocated to a subsequent 32-bit value.
    84:   CCIfType<[i64,v2i32,v4i16,v8i8],
    85:     CCCustom<"CC_SkipOdd">>,
    86:   CCIfType<[i64,v2i32,v4i16,v8i8],
    87:     CCAssignToReg<[D0,D1,D2]>>,
    88: 
    89:   CCDelegateTo<CC_HexagonStack>
    90: ]>;
    91: 
    92: def RetCC_Hexagon: CallingConv<[
    93:   CCIfType<[i1,i8,i16],
    94:     CCPromoteToType<i32>>,
    95:   CCIfType<[bf16],
    96:     CCBitConvertToType<i32>>,
    97:   CCIfType<[f32],
    98:     CCBitConvertToType<i32>>,
    99:   CCIfType<[f64],
   100:     CCBitConvertToType<i64>>,
```
- EN: It defines declarative TableGen records like RetCC_Hexagon; these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 RetCC_Hexagon 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 101-125 / 第 101-125 行

```tablegen
   101: 
   102:   // Small structures are returned in a pair of registers, (which is
   103:   // always r1:0). In such case, what is returned are two i32 values
   104:   // without any additional information (in ArgFlags) stating that
   105:   // they are parts of a structure. Because of that there is no way
   106:   // to differentiate that situation from an attempt to return two
   107:   // values, so always assign R0 and R1.
   108:   CCIfSplit<
   109:     CCAssignToReg<[R0,R1]>>,
   110:   CCIfType<[i32,v2i16,v4i8],
   111:     CCAssignToReg<[R0,R1]>>,
   112:   CCIfType<[i64,v2i32,v4i16,v8i8],
   113:     CCAssignToReg<[D0]>>
   114: ]>;
   115: 
   116: 
   117: class CCIfHvx64<CCAction A>
   118:   : CCIf<"State.getMachineFunction().getSubtarget<HexagonSubtarget>()"
   119:          ".useHVX64BOps()", A>;
   120: 
   121: class CCIfHvx128<CCAction A>
   122:   : CCIf<"State.getMachineFunction().getSubtarget<HexagonSubtarget>()"
   123:          ".useHVX128BOps()", A>;
   124: 
   125: def CC_Hexagon_HVX: CallingConv<[
```
- EN: It declares types such as CCIfHvx64, CCIfHvx128, which carry the state or API of this component. It defines declarative TableGen records like CCIfHvx64, CCIfHvx128, CC_Hexagon_HVX; these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明了 CCIfHvx64, CCIfHvx128 等类型，用来承载该组件的状态或接口。 这里定义了 CCIfHvx64, CCIfHvx128, CC_Hexagon_HVX 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 126-150 / 第 126-150 行

```tablegen
   126:   // HVX 64-byte mode
   127: 
   128:   CCIfHvx64<
   129:         CCIfType<[v16i1], CCPromoteToType<v16i32>>>,
   130:   CCIfHvx64<
   131:         CCIfType<[v32i1], CCPromoteToType<v32i16>>>,
   132:   CCIfHvx64<
   133:         CCIfType<[v64i1], CCPromoteToType<v64i8>>>,
   134: 
   135:   CCIfHvx64<
   136:     CCIfType<[v16i32,v32i16,v64i8],
   137:       CCAssignToReg<[V0,V1,V2,V3,V4,V5,V6,V7,V8,V9,V10,V11,V12,V13,V14,V15]>>>,
   138:   CCIfHvx64<
   139:     CCIfType<[v32i32,v64i16,v128i8],
   140:       CCAssignToReg<[W0,W1,W2,W3,W4,W5,W6,W7]>>>,
   141:   CCIfHvx64<
   142:     CCIfType<[v16i32,v32i16,v64i8],
   143:       CCAssignToStack<64,64>>>,
   144:   CCIfHvx64<
   145:     CCIfType<[v32i32,v64i16,v128i8],
   146:       CCAssignToStack<128,64>>>,
   147: 
   148:   // HVX 128-byte mode
   149: 
   150:   CCIfHvx128<
```
- EN: This range continues the declarative TableGen description for Hexagon target data.
- CN: 这一段继续给出 Hexagon 目标数据的声明式 TableGen 描述。

### Lines 151-175 / 第 151-175 行

```tablegen
   151:         CCIfType<[v32i1], CCPromoteToType<v32i32>>>,
   152:   CCIfHvx128<
   153:         CCIfType<[v64i1], CCPromoteToType<v64i16>>>,
   154:   CCIfHvx128<
   155:         CCIfType<[v128i1], CCPromoteToType<v128i8>>>,
   156: 
   157:   CCIfHvx128<
   158:     CCIfType<[v32i32,v64i16,v128i8,v32f32,v64f16,v64bf16],
   159:       CCAssignToReg<[V0,V1,V2,V3,V4,V5,V6,V7,V8,V9,V10,V11,V12,V13,V14,V15]>>>,
   160:   CCIfHvx128<
   161:     CCIfType<[v64i32,v128i16,v256i8,v64f32,v128f16,v128bf16],
   162:       CCAssignToReg<[W0,W1,W2,W3,W4,W5,W6,W7]>>>,
   163:   CCIfHvx128<
   164:     CCIfType<[v32i32,v64i16,v128i8,v32f32,v64f16,v64bf16],
   165:       CCAssignToStack<128,128>>>,
   166:   CCIfHvx128<
   167:     CCIfType<[v64i32,v128i16,v256i8,v64f32,v128f16,v64bf16],
   168:       CCAssignToStack<256,128>>>,
   169: 
   170:   CCDelegateTo<CC_Hexagon>
   171: ]>;
   172: 
   173: def RetCC_Hexagon_HVX: CallingConv<[
   174:   // HVX 64-byte mode
   175:   CCIfHvx64<
```
- EN: It defines declarative TableGen records like RetCC_Hexagon_HVX; these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 RetCC_Hexagon_HVX 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 176-192 / 第 176-192 行

```tablegen
   176:     CCIfType<[v16i32,v32i16,v64i8],
   177:       CCAssignToReg<[V0]>>>,
   178:   CCIfHvx64<
   179:     CCIfType<[v32i32,v64i16,v128i8],
   180:       CCAssignToReg<[W0]>>>,
   181: 
   182:   // HVX 128-byte mode
   183:   CCIfHvx128<
   184:     CCIfType<[v32i32,v64i16,v128i8,v32f32,v64f16,v64bf16],
   185:       CCAssignToReg<[V0]>>>,
   186:   CCIfHvx128<
   187:     CCIfType<[v64i32,v128i16,v256i8,v64f32,v128f16,v128bf16],
   188:       CCAssignToReg<[W0]>>>,
   189: 
   190:   CCDelegateTo<RetCC_Hexagon>
   191: ]>;
   192: 
```
- EN: This range continues the declarative TableGen description for Hexagon target data.
- CN: 这一段继续给出 Hexagon 目标数据的声明式 TableGen 描述。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- register modeling / 寄存器建模
- subtarget feature gating / 子目标特性控制
- calling convention encoding / 调用约定编码
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonCallingConv, HexagonSubtarget`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。

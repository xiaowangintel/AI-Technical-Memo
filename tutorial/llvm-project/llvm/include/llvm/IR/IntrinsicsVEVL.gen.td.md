# IntrinsicsVEVL.gen.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsVEVL.gen.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records, generated metadata, and declarative rules for `IntrinsicsVEVL.gen`.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsVEVL.gen` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vld_vssl : ClangBuiltin<"__builtin_ve_vl_vld_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vld_vssvl : ClangBuiltin<"__builtin_ve_vl_vld_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldnc_vssl : ClangBuiltin<"__builtin_ve_vl_vldnc_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldnc_vssvl : ClangBuiltin<"__builtin_ve_vl_vldnc_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldu_vssl : ClangBuiltin<"__builtin_ve_vl_vldu_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldu_vssvl : ClangBuiltin<"__builtin_ve_vl_vldu_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldunc_vssl : ClangBuiltin<"__builtin_ve_vl_vldunc_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldunc_vssvl : ClangBuiltin<"__builtin_ve_vl_vldunc_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldlsx_vssl : ClangBuiltin<"__builtin_ve_vl_vldlsx_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldlsx_vssvl : ClangBuiltin<"__builtin_ve_vl_vldlsx_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldlsxnc_vssl : ClangBuiltin<"__builtin_ve_vl_vldlsxnc_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldlsxnc_vssvl : ClangBuiltin<"__builtin_ve_vl_vldlsxnc_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldlzx_vssl : ClangBuiltin<"__builtin_ve_vl_vldlzx_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldlzx_vssvl : ClangBuiltin<"__builtin_ve_vl_vldlzx_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldlzxnc_vssl : ClangBuiltin<"__builtin_ve_vl_vldlzxnc_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldlzxnc_vssvl : ClangBuiltin<"__builtin_ve_vl_vldlzxnc_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vld2d_vssl : ClangBuiltin<"__builtin_ve_vl_vld2d_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vld2d_vssvl : ClangBuiltin<"__builtin_ve_vl_vld2d_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vld2dnc_vssl : ClangBuiltin<"__builtin_ve_vl_vld2dnc_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vld2dnc_vssvl : ClangBuiltin<"__builtin_ve_vl_vld2dnc_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldu2d_vssl : ClangBuiltin<"__builtin_ve_vl_vldu2d_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldu2d_vssvl : ClangBuiltin<"__builtin_ve_vl_vldu2d_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldu2dnc_vssl : ClangBuiltin<"__builtin_ve_vl_vldu2dnc_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldu2dnc_vssvl : ClangBuiltin<"__builtin_ve_vl_vldu2dnc_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
````
- **L1 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L3 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L4 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L4 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L5 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L5 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L6 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L6 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L7 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L7 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L8 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L8 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L9 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L9 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L10 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L10 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L11 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L11 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L12 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L12 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L13 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L13 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L14 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L14 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L15 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L15 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L16 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L16 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L17 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L17 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L18 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L18 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L19 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L19 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L20 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L20 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L21 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L21 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L22 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L22 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L23 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L23 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L24 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L24 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 25-48

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vldl2dsx_vssl : ClangBuiltin<"__builtin_ve_vl_vldl2dsx_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldl2dsx_vssvl : ClangBuiltin<"__builtin_ve_vl_vldl2dsx_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldl2dsxnc_vssl : ClangBuiltin<"__builtin_ve_vl_vldl2dsxnc_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldl2dsxnc_vssvl : ClangBuiltin<"__builtin_ve_vl_vldl2dsxnc_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldl2dzx_vssl : ClangBuiltin<"__builtin_ve_vl_vldl2dzx_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldl2dzx_vssvl : ClangBuiltin<"__builtin_ve_vl_vldl2dzx_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldl2dzxnc_vssl : ClangBuiltin<"__builtin_ve_vl_vldl2dzxnc_vssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldl2dzxnc_vssvl : ClangBuiltin<"__builtin_ve_vl_vldl2dzxnc_vssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<i64>, llvm_ptr_ty, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vst_vssl : ClangBuiltin<"__builtin_ve_vl_vst_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vst_vssml : ClangBuiltin<"__builtin_ve_vl_vst_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstnc_vssl : ClangBuiltin<"__builtin_ve_vl_vstnc_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstnc_vssml : ClangBuiltin<"__builtin_ve_vl_vstnc_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstot_vssl : ClangBuiltin<"__builtin_ve_vl_vstot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstot_vssml : ClangBuiltin<"__builtin_ve_vl_vstot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstncot_vssl : ClangBuiltin<"__builtin_ve_vl_vstncot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstncot_vssml : ClangBuiltin<"__builtin_ve_vl_vstncot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstu_vssl : ClangBuiltin<"__builtin_ve_vl_vstu_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstu_vssml : ClangBuiltin<"__builtin_ve_vl_vstu_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstunc_vssl : ClangBuiltin<"__builtin_ve_vl_vstunc_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstunc_vssml : ClangBuiltin<"__builtin_ve_vl_vstunc_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstuot_vssl : ClangBuiltin<"__builtin_ve_vl_vstuot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstuot_vssml : ClangBuiltin<"__builtin_ve_vl_vstuot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstuncot_vssl : ClangBuiltin<"__builtin_ve_vl_vstuncot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstuncot_vssml : ClangBuiltin<"__builtin_ve_vl_vstuncot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
````
- **L25 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L25 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L26 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L26 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L27 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L27 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L28 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L28 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L29 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L29 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L30 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L30 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L31 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L31 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L32 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L32 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L33 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L33 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L34 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L34 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L35 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L35 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L36 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L36 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L37 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L37 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L38 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L38 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L39 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L39 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L40 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L40 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L41 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L41 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L42 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L42 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L43 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L43 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L44 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L44 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L45 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L45 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L46 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L46 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L47 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L47 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L48 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L48 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 49-72

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vstl_vssl : ClangBuiltin<"__builtin_ve_vl_vstl_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstl_vssml : ClangBuiltin<"__builtin_ve_vl_vstl_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstlnc_vssl : ClangBuiltin<"__builtin_ve_vl_vstlnc_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstlnc_vssml : ClangBuiltin<"__builtin_ve_vl_vstlnc_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstlot_vssl : ClangBuiltin<"__builtin_ve_vl_vstlot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstlot_vssml : ClangBuiltin<"__builtin_ve_vl_vstlot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstlncot_vssl : ClangBuiltin<"__builtin_ve_vl_vstlncot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstlncot_vssml : ClangBuiltin<"__builtin_ve_vl_vstlncot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vst2d_vssl : ClangBuiltin<"__builtin_ve_vl_vst2d_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vst2d_vssml : ClangBuiltin<"__builtin_ve_vl_vst2d_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vst2dnc_vssl : ClangBuiltin<"__builtin_ve_vl_vst2dnc_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vst2dnc_vssml : ClangBuiltin<"__builtin_ve_vl_vst2dnc_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vst2dot_vssl : ClangBuiltin<"__builtin_ve_vl_vst2dot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vst2dot_vssml : ClangBuiltin<"__builtin_ve_vl_vst2dot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vst2dncot_vssl : ClangBuiltin<"__builtin_ve_vl_vst2dncot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vst2dncot_vssml : ClangBuiltin<"__builtin_ve_vl_vst2dncot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstu2d_vssl : ClangBuiltin<"__builtin_ve_vl_vstu2d_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstu2d_vssml : ClangBuiltin<"__builtin_ve_vl_vstu2d_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstu2dnc_vssl : ClangBuiltin<"__builtin_ve_vl_vstu2dnc_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstu2dnc_vssml : ClangBuiltin<"__builtin_ve_vl_vstu2dnc_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstu2dot_vssl : ClangBuiltin<"__builtin_ve_vl_vstu2dot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstu2dot_vssml : ClangBuiltin<"__builtin_ve_vl_vstu2dot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstu2dncot_vssl : ClangBuiltin<"__builtin_ve_vl_vstu2dncot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstu2dncot_vssml : ClangBuiltin<"__builtin_ve_vl_vstu2dncot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
````
- **L49 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L49 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L50 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L50 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L51 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L51 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L52 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L52 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L53 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L53 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L54 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L54 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L55 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L55 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L56 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L56 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L57 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L57 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L58 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L58 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L59 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L59 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L60 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L60 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L61 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L61 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L62 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L62 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L63 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L63 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L64 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L64 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L65 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L65 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L66 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L66 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L67 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L67 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L68 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L68 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L69 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L69 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L70 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L70 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L71 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L71 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L72 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L72 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 73-96

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vstl2d_vssl : ClangBuiltin<"__builtin_ve_vl_vstl2d_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstl2d_vssml : ClangBuiltin<"__builtin_ve_vl_vstl2d_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstl2dnc_vssl : ClangBuiltin<"__builtin_ve_vl_vstl2dnc_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstl2dnc_vssml : ClangBuiltin<"__builtin_ve_vl_vstl2dnc_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstl2dot_vssl : ClangBuiltin<"__builtin_ve_vl_vstl2dot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstl2dot_vssml : ClangBuiltin<"__builtin_ve_vl_vstl2dot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstl2dncot_vssl : ClangBuiltin<"__builtin_ve_vl_vstl2dncot_vssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vstl2dncot_vssml : ClangBuiltin<"__builtin_ve_vl_vstl2dncot_vssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<i64>, llvm_ptr_ty, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pfchv_ssl : ClangBuiltin<"__builtin_ve_vl_pfchv_ssl">, Intrinsic<[], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrInaccessibleMemOrArgMemOnly]>;
let TargetPrefix = "ve" in def int_ve_vl_pfchvnc_ssl : ClangBuiltin<"__builtin_ve_vl_pfchvnc_ssl">, Intrinsic<[], [LLVMType<i64>, llvm_ptr_ty, LLVMType<i32>], [IntrInaccessibleMemOrArgMemOnly]>;
let TargetPrefix = "ve" in def int_ve_vl_lsv_vvss : ClangBuiltin<"__builtin_ve_vl_lsv_vvss">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<i64>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_lvsl_svs : ClangBuiltin<"__builtin_ve_vl_lvsl_svs">, Intrinsic<[LLVMType<i64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_lvsd_svs : ClangBuiltin<"__builtin_ve_vl_lvsd_svs">, Intrinsic<[LLVMType<f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_lvss_svs : ClangBuiltin<"__builtin_ve_vl_lvss_svs">, Intrinsic<[LLVMType<f32>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_lvm_mmss : ClangBuiltin<"__builtin_ve_vl_lvm_mmss">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256i1>, LLVMType<i64>, LLVMType<i64>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_lvm_MMss : ClangBuiltin<"__builtin_ve_vl_lvm_MMss">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v512i1>, LLVMType<i64>, LLVMType<i64>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_svm_sms : ClangBuiltin<"__builtin_ve_vl_svm_sms">, Intrinsic<[LLVMType<i64>], [LLVMType<v256i1>, LLVMType<i64>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_svm_sMs : ClangBuiltin<"__builtin_ve_vl_svm_sMs">, Intrinsic<[LLVMType<i64>], [LLVMType<v512i1>, LLVMType<i64>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrdd_vsl : ClangBuiltin<"__builtin_ve_vl_vbrdd_vsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrdd_vsvl : ClangBuiltin<"__builtin_ve_vl_vbrdd_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrdd_vsmvl : ClangBuiltin<"__builtin_ve_vl_vbrdd_vsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrdl_vsl : ClangBuiltin<"__builtin_ve_vl_vbrdl_vsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrdl_vsvl : ClangBuiltin<"__builtin_ve_vl_vbrdl_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrdl_vsmvl : ClangBuiltin<"__builtin_ve_vl_vbrdl_vsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L73 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L73 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L74 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L74 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L75 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L75 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L76 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L76 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L77 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L77 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L78 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L78 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L79 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L79 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L80 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L80 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L81 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L81 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L82 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L82 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L83 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L83 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L84 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L84 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L85 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L85 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L86 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L86 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L87 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L87 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L88 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L88 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L89 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L89 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L90 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L90 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L91 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L91 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L92 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L92 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L93 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L93 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L94 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L94 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L95 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L95 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L96 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L96 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 97-120

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vbrds_vsl : ClangBuiltin<"__builtin_ve_vl_vbrds_vsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrds_vsvl : ClangBuiltin<"__builtin_ve_vl_vbrds_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrds_vsmvl : ClangBuiltin<"__builtin_ve_vl_vbrds_vsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrdw_vsl : ClangBuiltin<"__builtin_ve_vl_vbrdw_vsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrdw_vsvl : ClangBuiltin<"__builtin_ve_vl_vbrdw_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrdw_vsmvl : ClangBuiltin<"__builtin_ve_vl_vbrdw_vsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrd_vsl : ClangBuiltin<"__builtin_ve_vl_pvbrd_vsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrd_vsvl : ClangBuiltin<"__builtin_ve_vl_pvbrd_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrd_vsMvl : ClangBuiltin<"__builtin_ve_vl_pvbrd_vsMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmv_vsvl : ClangBuiltin<"__builtin_ve_vl_vmv_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmv_vsvvl : ClangBuiltin<"__builtin_ve_vl_vmv_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmv_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vmv_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddul_vvvl : ClangBuiltin<"__builtin_ve_vl_vaddul_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddul_vvvvl : ClangBuiltin<"__builtin_ve_vl_vaddul_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddul_vsvl : ClangBuiltin<"__builtin_ve_vl_vaddul_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddul_vsvvl : ClangBuiltin<"__builtin_ve_vl_vaddul_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddul_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vaddul_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddul_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vaddul_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vadduw_vvvl : ClangBuiltin<"__builtin_ve_vl_vadduw_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vadduw_vvvvl : ClangBuiltin<"__builtin_ve_vl_vadduw_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vadduw_vsvl : ClangBuiltin<"__builtin_ve_vl_vadduw_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vadduw_vsvvl : ClangBuiltin<"__builtin_ve_vl_vadduw_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vadduw_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vadduw_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vadduw_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vadduw_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L97 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L97 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L98 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L98 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L99 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L99 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L100 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L100 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L101 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L101 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L102 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L102 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L103 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L103 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L104 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L104 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L105 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L105 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L106 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L106 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L107 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L107 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L108 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L108 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L109 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L109 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L110 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L110 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L111 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L111 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L112 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L112 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L113 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L113 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L114 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L114 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L115 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L115 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L116 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L116 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L117 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L117 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L118 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L118 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L119 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L119 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L120 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L120 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 121-144

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvaddu_vvvl : ClangBuiltin<"__builtin_ve_vl_pvaddu_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvaddu_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvaddu_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvaddu_vsvl : ClangBuiltin<"__builtin_ve_vl_pvaddu_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvaddu_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvaddu_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvaddu_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvaddu_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvaddu_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvaddu_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vaddswsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswsx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vaddswsx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswsx_vsvl : ClangBuiltin<"__builtin_ve_vl_vaddswsx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswsx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vaddswsx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswsx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vaddswsx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswsx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vaddswsx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vaddswzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswzx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vaddswzx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswzx_vsvl : ClangBuiltin<"__builtin_ve_vl_vaddswzx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswzx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vaddswzx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswzx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vaddswzx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddswzx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vaddswzx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvadds_vvvl : ClangBuiltin<"__builtin_ve_vl_pvadds_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvadds_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvadds_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvadds_vsvl : ClangBuiltin<"__builtin_ve_vl_pvadds_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvadds_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvadds_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvadds_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvadds_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvadds_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvadds_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L121 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L121 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L122 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L122 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L123 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L123 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L124 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L124 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L125 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L125 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L126 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L126 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L127 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L127 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L128 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L128 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L129 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L129 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L130 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L130 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L131 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L131 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L132 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L132 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L133 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L133 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L134 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L134 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L135 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L135 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L136 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L136 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L137 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L137 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L138 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L138 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L139 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L139 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L140 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L140 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L141 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L141 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L142 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L142 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L143 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L143 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L144 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L144 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 145-168

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vaddsl_vvvl : ClangBuiltin<"__builtin_ve_vl_vaddsl_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddsl_vvvvl : ClangBuiltin<"__builtin_ve_vl_vaddsl_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddsl_vsvl : ClangBuiltin<"__builtin_ve_vl_vaddsl_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddsl_vsvvl : ClangBuiltin<"__builtin_ve_vl_vaddsl_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddsl_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vaddsl_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vaddsl_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vaddsl_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubul_vvvl : ClangBuiltin<"__builtin_ve_vl_vsubul_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubul_vvvvl : ClangBuiltin<"__builtin_ve_vl_vsubul_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubul_vsvl : ClangBuiltin<"__builtin_ve_vl_vsubul_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubul_vsvvl : ClangBuiltin<"__builtin_ve_vl_vsubul_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubul_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vsubul_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubul_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vsubul_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubuw_vvvl : ClangBuiltin<"__builtin_ve_vl_vsubuw_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubuw_vvvvl : ClangBuiltin<"__builtin_ve_vl_vsubuw_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubuw_vsvl : ClangBuiltin<"__builtin_ve_vl_vsubuw_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubuw_vsvvl : ClangBuiltin<"__builtin_ve_vl_vsubuw_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubuw_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vsubuw_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubuw_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vsubuw_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubu_vvvl : ClangBuiltin<"__builtin_ve_vl_pvsubu_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubu_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvsubu_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubu_vsvl : ClangBuiltin<"__builtin_ve_vl_pvsubu_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubu_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvsubu_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubu_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvsubu_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubu_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvsubu_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L145 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L145 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L146 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L146 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L147 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L147 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L148 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L148 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L149 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L149 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L150 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L150 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L151 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L151 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L152 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L152 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L153 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L153 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L154 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L154 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L155 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L155 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L156 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L156 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L157 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L157 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L158 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L158 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L159 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L159 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L160 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L160 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L161 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L161 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L162 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L162 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L163 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L163 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L164 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L164 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L165 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L165 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L166 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L166 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L167 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L167 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L168 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L168 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 169-192

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vsubswsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vsubswsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubswsx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vsubswsx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubswsx_vsvl : ClangBuiltin<"__builtin_ve_vl_vsubswsx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubswsx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vsubswsx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubswsx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vsubswsx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubswsx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vsubswsx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubswzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vsubswzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubswzx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vsubswzx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubswzx_vsvl : ClangBuiltin<"__builtin_ve_vl_vsubswzx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubswzx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vsubswzx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubswzx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vsubswzx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubswzx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vsubswzx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubs_vvvl : ClangBuiltin<"__builtin_ve_vl_pvsubs_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubs_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvsubs_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubs_vsvl : ClangBuiltin<"__builtin_ve_vl_pvsubs_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubs_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvsubs_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubs_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvsubs_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsubs_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvsubs_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubsl_vvvl : ClangBuiltin<"__builtin_ve_vl_vsubsl_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubsl_vvvvl : ClangBuiltin<"__builtin_ve_vl_vsubsl_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubsl_vsvl : ClangBuiltin<"__builtin_ve_vl_vsubsl_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubsl_vsvvl : ClangBuiltin<"__builtin_ve_vl_vsubsl_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubsl_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vsubsl_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsubsl_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vsubsl_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L169 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L169 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L170 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L170 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L171 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L171 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L172 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L172 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L173 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L173 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L174 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L174 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L175 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L175 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L176 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L176 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L177 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L177 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L178 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L178 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L179 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L179 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L180 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L180 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L181 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L181 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L182 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L182 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L183 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L183 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L184 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L184 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L185 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L185 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L186 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L186 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L187 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L187 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L188 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L188 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L189 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L189 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L190 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L190 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L191 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L191 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L192 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L192 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 193-216

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vmulul_vvvl : ClangBuiltin<"__builtin_ve_vl_vmulul_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulul_vvvvl : ClangBuiltin<"__builtin_ve_vl_vmulul_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulul_vsvl : ClangBuiltin<"__builtin_ve_vl_vmulul_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulul_vsvvl : ClangBuiltin<"__builtin_ve_vl_vmulul_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulul_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vmulul_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulul_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vmulul_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmuluw_vvvl : ClangBuiltin<"__builtin_ve_vl_vmuluw_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmuluw_vvvvl : ClangBuiltin<"__builtin_ve_vl_vmuluw_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmuluw_vsvl : ClangBuiltin<"__builtin_ve_vl_vmuluw_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmuluw_vsvvl : ClangBuiltin<"__builtin_ve_vl_vmuluw_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmuluw_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vmuluw_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmuluw_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vmuluw_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vmulswsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswsx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vmulswsx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswsx_vsvl : ClangBuiltin<"__builtin_ve_vl_vmulswsx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswsx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vmulswsx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswsx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vmulswsx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswsx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vmulswsx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vmulswzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswzx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vmulswzx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswzx_vsvl : ClangBuiltin<"__builtin_ve_vl_vmulswzx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswzx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vmulswzx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswzx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vmulswzx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulswzx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vmulswzx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L193 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L193 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L194 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L194 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L195 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L195 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L196 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L196 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L197 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L197 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L198 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L198 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L199 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L199 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L200 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L200 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L201 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L201 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L202 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L202 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L203 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L203 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L204 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L204 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L205 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L205 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L206 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L206 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L207 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L207 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L208 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L208 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L209 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L209 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L210 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L210 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L211 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L211 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L212 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L212 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L213 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L213 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L214 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L214 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L215 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L215 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L216 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L216 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 217-240

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vmulsl_vvvl : ClangBuiltin<"__builtin_ve_vl_vmulsl_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulsl_vvvvl : ClangBuiltin<"__builtin_ve_vl_vmulsl_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulsl_vsvl : ClangBuiltin<"__builtin_ve_vl_vmulsl_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulsl_vsvvl : ClangBuiltin<"__builtin_ve_vl_vmulsl_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulsl_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vmulsl_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulsl_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vmulsl_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulslw_vvvl : ClangBuiltin<"__builtin_ve_vl_vmulslw_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulslw_vvvvl : ClangBuiltin<"__builtin_ve_vl_vmulslw_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulslw_vsvl : ClangBuiltin<"__builtin_ve_vl_vmulslw_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmulslw_vsvvl : ClangBuiltin<"__builtin_ve_vl_vmulslw_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivul_vvvl : ClangBuiltin<"__builtin_ve_vl_vdivul_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivul_vvvvl : ClangBuiltin<"__builtin_ve_vl_vdivul_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivul_vsvl : ClangBuiltin<"__builtin_ve_vl_vdivul_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivul_vsvvl : ClangBuiltin<"__builtin_ve_vl_vdivul_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivul_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vdivul_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivul_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vdivul_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivuw_vvvl : ClangBuiltin<"__builtin_ve_vl_vdivuw_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivuw_vvvvl : ClangBuiltin<"__builtin_ve_vl_vdivuw_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivuw_vsvl : ClangBuiltin<"__builtin_ve_vl_vdivuw_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivuw_vsvvl : ClangBuiltin<"__builtin_ve_vl_vdivuw_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivuw_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vdivuw_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivuw_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vdivuw_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivul_vvsl : ClangBuiltin<"__builtin_ve_vl_vdivul_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivul_vvsvl : ClangBuiltin<"__builtin_ve_vl_vdivul_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L217 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L217 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L218 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L218 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L219 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L219 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L220 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L220 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L221 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L221 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L222 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L222 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L223 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L223 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L224 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L224 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L225 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L225 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L226 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L226 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L227 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L227 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L228 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L228 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L229 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L229 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L230 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L230 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L231 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L231 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L232 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L232 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L233 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L233 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L234 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L234 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L235 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L235 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L236 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L236 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L237 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L237 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L238 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L238 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L239 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L239 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L240 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L240 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 241-264

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vdivul_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vdivul_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivuw_vvsl : ClangBuiltin<"__builtin_ve_vl_vdivuw_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivuw_vvsvl : ClangBuiltin<"__builtin_ve_vl_vdivuw_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivuw_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vdivuw_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vdivswsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswsx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vdivswsx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswsx_vsvl : ClangBuiltin<"__builtin_ve_vl_vdivswsx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswsx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vdivswsx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswsx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vdivswsx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswsx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vdivswsx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vdivswzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswzx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vdivswzx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswzx_vsvl : ClangBuiltin<"__builtin_ve_vl_vdivswzx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswzx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vdivswzx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswzx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vdivswzx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswzx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vdivswzx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswsx_vvsl : ClangBuiltin<"__builtin_ve_vl_vdivswsx_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswsx_vvsvl : ClangBuiltin<"__builtin_ve_vl_vdivswsx_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswsx_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vdivswsx_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswzx_vvsl : ClangBuiltin<"__builtin_ve_vl_vdivswzx_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswzx_vvsvl : ClangBuiltin<"__builtin_ve_vl_vdivswzx_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivswzx_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vdivswzx_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivsl_vvvl : ClangBuiltin<"__builtin_ve_vl_vdivsl_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivsl_vvvvl : ClangBuiltin<"__builtin_ve_vl_vdivsl_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L241 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L241 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L242 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L242 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L243 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L243 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L244 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L244 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L245 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L245 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L246 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L246 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L247 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L247 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L248 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L248 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L249 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L249 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L250 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L250 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L251 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L251 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L252 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L252 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L253 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L253 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L254 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L254 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L255 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L255 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L256 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L256 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L257 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L257 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L258 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L258 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L259 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L259 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L260 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L260 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L261 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L261 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L262 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L262 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L263 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L263 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L264 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L264 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 265-288

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vdivsl_vsvl : ClangBuiltin<"__builtin_ve_vl_vdivsl_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivsl_vsvvl : ClangBuiltin<"__builtin_ve_vl_vdivsl_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivsl_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vdivsl_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivsl_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vdivsl_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivsl_vvsl : ClangBuiltin<"__builtin_ve_vl_vdivsl_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivsl_vvsvl : ClangBuiltin<"__builtin_ve_vl_vdivsl_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vdivsl_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vdivsl_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpul_vvvl : ClangBuiltin<"__builtin_ve_vl_vcmpul_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpul_vvvvl : ClangBuiltin<"__builtin_ve_vl_vcmpul_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpul_vsvl : ClangBuiltin<"__builtin_ve_vl_vcmpul_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpul_vsvvl : ClangBuiltin<"__builtin_ve_vl_vcmpul_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpul_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vcmpul_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpul_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vcmpul_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpuw_vvvl : ClangBuiltin<"__builtin_ve_vl_vcmpuw_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpuw_vvvvl : ClangBuiltin<"__builtin_ve_vl_vcmpuw_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpuw_vsvl : ClangBuiltin<"__builtin_ve_vl_vcmpuw_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpuw_vsvvl : ClangBuiltin<"__builtin_ve_vl_vcmpuw_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpuw_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vcmpuw_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpuw_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vcmpuw_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcmpu_vvvl : ClangBuiltin<"__builtin_ve_vl_pvcmpu_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcmpu_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvcmpu_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcmpu_vsvl : ClangBuiltin<"__builtin_ve_vl_pvcmpu_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcmpu_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvcmpu_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcmpu_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvcmpu_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L265 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L265 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L266 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L266 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L267 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L267 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L268 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L268 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L269 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L269 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L270 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L270 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L271 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L271 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L272 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L272 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L273 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L273 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L274 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L274 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L275 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L275 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L276 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L276 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L277 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L277 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L278 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L278 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L279 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L279 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L280 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L280 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L281 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L281 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L282 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L282 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L283 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L283 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L284 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L284 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L285 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L285 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L286 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L286 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L287 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L287 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L288 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L288 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 289-312

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvcmpu_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvcmpu_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vcmpswsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswsx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vcmpswsx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswsx_vsvl : ClangBuiltin<"__builtin_ve_vl_vcmpswsx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswsx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vcmpswsx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswsx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vcmpswsx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswsx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vcmpswsx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vcmpswzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswzx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vcmpswzx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswzx_vsvl : ClangBuiltin<"__builtin_ve_vl_vcmpswzx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswzx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vcmpswzx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswzx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vcmpswzx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpswzx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vcmpswzx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcmps_vvvl : ClangBuiltin<"__builtin_ve_vl_pvcmps_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcmps_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvcmps_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcmps_vsvl : ClangBuiltin<"__builtin_ve_vl_pvcmps_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcmps_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvcmps_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcmps_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvcmps_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcmps_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvcmps_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpsl_vvvl : ClangBuiltin<"__builtin_ve_vl_vcmpsl_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpsl_vvvvl : ClangBuiltin<"__builtin_ve_vl_vcmpsl_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpsl_vsvl : ClangBuiltin<"__builtin_ve_vl_vcmpsl_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpsl_vsvvl : ClangBuiltin<"__builtin_ve_vl_vcmpsl_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcmpsl_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vcmpsl_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L289 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L289 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L290 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L290 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L291 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L291 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L292 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L292 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L293 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L293 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L294 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L294 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L295 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L295 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L296 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L296 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L297 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L297 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L298 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L298 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L299 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L299 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L300 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L300 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L301 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L301 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L302 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L302 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L303 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L303 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L304 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L304 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L305 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L305 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L306 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L306 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L307 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L307 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L308 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L308 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L309 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L309 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L310 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L310 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L311 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L311 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L312 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L312 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 313-336

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vcmpsl_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vcmpsl_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vmaxswsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswsx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vmaxswsx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswsx_vsvl : ClangBuiltin<"__builtin_ve_vl_vmaxswsx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswsx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vmaxswsx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswsx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vmaxswsx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswsx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vmaxswsx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vmaxswzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswzx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vmaxswzx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswzx_vsvl : ClangBuiltin<"__builtin_ve_vl_vmaxswzx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswzx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vmaxswzx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswzx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vmaxswzx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxswzx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vmaxswzx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmaxs_vvvl : ClangBuiltin<"__builtin_ve_vl_pvmaxs_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmaxs_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvmaxs_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmaxs_vsvl : ClangBuiltin<"__builtin_ve_vl_pvmaxs_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmaxs_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvmaxs_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmaxs_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvmaxs_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmaxs_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvmaxs_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminswsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vminswsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminswsx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vminswsx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminswsx_vsvl : ClangBuiltin<"__builtin_ve_vl_vminswsx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminswsx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vminswsx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminswsx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vminswsx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L313 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L313 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L314 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L314 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L315 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L315 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L316 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L316 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L317 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L317 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L318 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L318 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L319 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L319 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L320 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L320 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L321 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L321 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L322 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L322 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L323 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L323 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L324 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L324 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L325 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L325 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L326 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L326 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L327 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L327 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L328 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L328 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L329 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L329 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L330 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L330 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L331 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L331 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L332 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L332 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L333 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L333 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L334 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L334 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L335 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L335 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L336 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L336 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 337-360

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vminswsx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vminswsx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminswzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vminswzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminswzx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vminswzx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminswzx_vsvl : ClangBuiltin<"__builtin_ve_vl_vminswzx_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminswzx_vsvvl : ClangBuiltin<"__builtin_ve_vl_vminswzx_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminswzx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vminswzx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminswzx_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vminswzx_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmins_vvvl : ClangBuiltin<"__builtin_ve_vl_pvmins_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmins_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvmins_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmins_vsvl : ClangBuiltin<"__builtin_ve_vl_pvmins_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmins_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvmins_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmins_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvmins_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvmins_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvmins_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxsl_vvvl : ClangBuiltin<"__builtin_ve_vl_vmaxsl_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxsl_vvvvl : ClangBuiltin<"__builtin_ve_vl_vmaxsl_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxsl_vsvl : ClangBuiltin<"__builtin_ve_vl_vmaxsl_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxsl_vsvvl : ClangBuiltin<"__builtin_ve_vl_vmaxsl_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxsl_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vmaxsl_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmaxsl_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vmaxsl_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminsl_vvvl : ClangBuiltin<"__builtin_ve_vl_vminsl_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminsl_vvvvl : ClangBuiltin<"__builtin_ve_vl_vminsl_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminsl_vsvl : ClangBuiltin<"__builtin_ve_vl_vminsl_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminsl_vsvvl : ClangBuiltin<"__builtin_ve_vl_vminsl_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vminsl_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vminsl_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L337 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L337 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L338 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L338 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L339 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L339 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L340 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L340 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L341 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L341 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L342 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L342 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L343 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L343 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L344 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L344 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L345 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L345 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L346 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L346 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L347 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L347 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L348 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L348 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L349 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L349 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L350 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L350 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L351 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L351 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L352 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L352 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L353 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L353 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L354 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L354 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L355 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L355 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L356 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L356 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L357 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L357 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L358 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L358 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L359 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L359 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L360 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L360 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 361-384

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vminsl_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vminsl_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vand_vvvl : ClangBuiltin<"__builtin_ve_vl_vand_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vand_vvvvl : ClangBuiltin<"__builtin_ve_vl_vand_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vand_vsvl : ClangBuiltin<"__builtin_ve_vl_vand_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vand_vsvvl : ClangBuiltin<"__builtin_ve_vl_vand_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vand_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vand_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vand_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vand_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvand_vvvl : ClangBuiltin<"__builtin_ve_vl_pvand_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvand_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvand_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvand_vsvl : ClangBuiltin<"__builtin_ve_vl_pvand_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvand_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvand_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvand_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvand_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvand_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvand_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vor_vvvl : ClangBuiltin<"__builtin_ve_vl_vor_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vor_vvvvl : ClangBuiltin<"__builtin_ve_vl_vor_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vor_vsvl : ClangBuiltin<"__builtin_ve_vl_vor_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vor_vsvvl : ClangBuiltin<"__builtin_ve_vl_vor_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vor_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vor_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vor_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vor_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvor_vvvl : ClangBuiltin<"__builtin_ve_vl_pvor_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvor_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvor_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvor_vsvl : ClangBuiltin<"__builtin_ve_vl_pvor_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvor_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvor_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvor_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvor_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L361 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L361 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L362 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L362 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L363 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L363 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L364 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L364 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L365 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L365 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L366 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L366 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L367 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L367 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L368 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L368 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L369 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L369 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L370 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L370 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L371 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L371 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L372 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L372 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L373 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L373 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L374 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L374 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L375 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L375 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L376 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L376 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L377 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L377 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L378 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L378 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L379 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L379 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L380 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L380 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L381 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L381 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L382 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L382 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L383 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L383 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L384 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L384 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 385-408

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvor_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvor_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vxor_vvvl : ClangBuiltin<"__builtin_ve_vl_vxor_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vxor_vvvvl : ClangBuiltin<"__builtin_ve_vl_vxor_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vxor_vsvl : ClangBuiltin<"__builtin_ve_vl_vxor_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vxor_vsvvl : ClangBuiltin<"__builtin_ve_vl_vxor_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vxor_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vxor_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vxor_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vxor_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvxor_vvvl : ClangBuiltin<"__builtin_ve_vl_pvxor_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvxor_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvxor_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvxor_vsvl : ClangBuiltin<"__builtin_ve_vl_pvxor_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvxor_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvxor_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvxor_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvxor_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvxor_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvxor_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_veqv_vvvl : ClangBuiltin<"__builtin_ve_vl_veqv_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_veqv_vvvvl : ClangBuiltin<"__builtin_ve_vl_veqv_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_veqv_vsvl : ClangBuiltin<"__builtin_ve_vl_veqv_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_veqv_vsvvl : ClangBuiltin<"__builtin_ve_vl_veqv_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_veqv_vvvmvl : ClangBuiltin<"__builtin_ve_vl_veqv_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_veqv_vsvmvl : ClangBuiltin<"__builtin_ve_vl_veqv_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pveqv_vvvl : ClangBuiltin<"__builtin_ve_vl_pveqv_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pveqv_vvvvl : ClangBuiltin<"__builtin_ve_vl_pveqv_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pveqv_vsvl : ClangBuiltin<"__builtin_ve_vl_pveqv_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pveqv_vsvvl : ClangBuiltin<"__builtin_ve_vl_pveqv_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pveqv_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pveqv_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L385 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L385 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L386 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L386 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L387 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L387 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L388 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L388 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L389 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L389 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L390 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L390 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L391 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L391 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L392 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L392 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L393 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L393 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L394 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L394 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L395 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L395 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L396 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L396 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L397 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L397 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L398 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L398 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L399 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L399 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L400 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L400 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L401 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L401 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L402 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L402 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L403 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L403 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L404 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L404 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L405 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L405 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L406 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L406 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L407 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L407 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L408 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L408 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 409-432

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pveqv_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pveqv_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldz_vvl : ClangBuiltin<"__builtin_ve_vl_vldz_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldz_vvvl : ClangBuiltin<"__builtin_ve_vl_vldz_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vldz_vvmvl : ClangBuiltin<"__builtin_ve_vl_vldz_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvldzlo_vvl : ClangBuiltin<"__builtin_ve_vl_pvldzlo_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvldzlo_vvvl : ClangBuiltin<"__builtin_ve_vl_pvldzlo_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvldzlo_vvmvl : ClangBuiltin<"__builtin_ve_vl_pvldzlo_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvldzup_vvl : ClangBuiltin<"__builtin_ve_vl_pvldzup_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvldzup_vvvl : ClangBuiltin<"__builtin_ve_vl_pvldzup_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvldzup_vvmvl : ClangBuiltin<"__builtin_ve_vl_pvldzup_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvldz_vvl : ClangBuiltin<"__builtin_ve_vl_pvldz_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvldz_vvvl : ClangBuiltin<"__builtin_ve_vl_pvldz_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvldz_vvMvl : ClangBuiltin<"__builtin_ve_vl_pvldz_vvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vpcnt_vvl : ClangBuiltin<"__builtin_ve_vl_vpcnt_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vpcnt_vvvl : ClangBuiltin<"__builtin_ve_vl_vpcnt_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vpcnt_vvmvl : ClangBuiltin<"__builtin_ve_vl_vpcnt_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvpcntlo_vvl : ClangBuiltin<"__builtin_ve_vl_pvpcntlo_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvpcntlo_vvvl : ClangBuiltin<"__builtin_ve_vl_pvpcntlo_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvpcntlo_vvmvl : ClangBuiltin<"__builtin_ve_vl_pvpcntlo_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvpcntup_vvl : ClangBuiltin<"__builtin_ve_vl_pvpcntup_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvpcntup_vvvl : ClangBuiltin<"__builtin_ve_vl_pvpcntup_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvpcntup_vvmvl : ClangBuiltin<"__builtin_ve_vl_pvpcntup_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvpcnt_vvl : ClangBuiltin<"__builtin_ve_vl_pvpcnt_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvpcnt_vvvl : ClangBuiltin<"__builtin_ve_vl_pvpcnt_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L409 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L409 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L410 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L410 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L411 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L411 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L412 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L412 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L413 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L413 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L414 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L414 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L415 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L415 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L416 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L416 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L417 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L417 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L418 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L418 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L419 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L419 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L420 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L420 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L421 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L421 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L422 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L422 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L423 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L423 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L424 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L424 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L425 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L425 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L426 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L426 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L427 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L427 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L428 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L428 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L429 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L429 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L430 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L430 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L431 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L431 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L432 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L432 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 433-456

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvpcnt_vvMvl : ClangBuiltin<"__builtin_ve_vl_pvpcnt_vvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrv_vvl : ClangBuiltin<"__builtin_ve_vl_vbrv_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrv_vvvl : ClangBuiltin<"__builtin_ve_vl_vbrv_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vbrv_vvmvl : ClangBuiltin<"__builtin_ve_vl_vbrv_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrvlo_vvl : ClangBuiltin<"__builtin_ve_vl_pvbrvlo_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrvlo_vvvl : ClangBuiltin<"__builtin_ve_vl_pvbrvlo_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrvlo_vvmvl : ClangBuiltin<"__builtin_ve_vl_pvbrvlo_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrvup_vvl : ClangBuiltin<"__builtin_ve_vl_pvbrvup_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrvup_vvvl : ClangBuiltin<"__builtin_ve_vl_pvbrvup_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrvup_vvmvl : ClangBuiltin<"__builtin_ve_vl_pvbrvup_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrv_vvl : ClangBuiltin<"__builtin_ve_vl_pvbrv_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrv_vvvl : ClangBuiltin<"__builtin_ve_vl_pvbrv_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvbrv_vvMvl : ClangBuiltin<"__builtin_ve_vl_pvbrv_vvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vseq_vl : ClangBuiltin<"__builtin_ve_vl_vseq_vl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vseq_vvl : ClangBuiltin<"__builtin_ve_vl_vseq_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvseqlo_vl : ClangBuiltin<"__builtin_ve_vl_pvseqlo_vl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvseqlo_vvl : ClangBuiltin<"__builtin_ve_vl_pvseqlo_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsequp_vl : ClangBuiltin<"__builtin_ve_vl_pvsequp_vl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsequp_vvl : ClangBuiltin<"__builtin_ve_vl_pvsequp_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvseq_vl : ClangBuiltin<"__builtin_ve_vl_pvseq_vl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvseq_vvl : ClangBuiltin<"__builtin_ve_vl_pvseq_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsll_vvvl : ClangBuiltin<"__builtin_ve_vl_vsll_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsll_vvvvl : ClangBuiltin<"__builtin_ve_vl_vsll_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsll_vvsl : ClangBuiltin<"__builtin_ve_vl_vsll_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L433 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L433 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L434 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L434 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L435 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L435 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L436 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L436 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L437 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L437 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L438 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L438 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L439 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L439 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L440 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L440 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L441 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L441 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L442 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L442 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L443 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L443 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L444 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L444 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L445 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L445 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L446 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L446 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L447 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L447 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L448 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L448 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L449 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L449 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L450 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L450 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L451 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L451 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L452 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L452 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L453 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L453 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L454 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L454 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L455 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L455 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L456 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L456 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 457-480

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vsll_vvsvl : ClangBuiltin<"__builtin_ve_vl_vsll_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsll_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vsll_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsll_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vsll_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsll_vvvl : ClangBuiltin<"__builtin_ve_vl_pvsll_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsll_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvsll_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsll_vvsl : ClangBuiltin<"__builtin_ve_vl_pvsll_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsll_vvsvl : ClangBuiltin<"__builtin_ve_vl_pvsll_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsll_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvsll_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsll_vvsMvl : ClangBuiltin<"__builtin_ve_vl_pvsll_vvsMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrl_vvvl : ClangBuiltin<"__builtin_ve_vl_vsrl_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrl_vvvvl : ClangBuiltin<"__builtin_ve_vl_vsrl_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrl_vvsl : ClangBuiltin<"__builtin_ve_vl_vsrl_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrl_vvsvl : ClangBuiltin<"__builtin_ve_vl_vsrl_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrl_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vsrl_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrl_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vsrl_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsrl_vvvl : ClangBuiltin<"__builtin_ve_vl_pvsrl_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsrl_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvsrl_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsrl_vvsl : ClangBuiltin<"__builtin_ve_vl_pvsrl_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsrl_vvsvl : ClangBuiltin<"__builtin_ve_vl_pvsrl_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsrl_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvsrl_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsrl_vvsMvl : ClangBuiltin<"__builtin_ve_vl_pvsrl_vvsMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslawsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vslawsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslawsx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vslawsx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslawsx_vvsl : ClangBuiltin<"__builtin_ve_vl_vslawsx_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<i32>], [IntrNoMem]>;
````
- **L457 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L457 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L458 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L458 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L459 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L459 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L460 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L460 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L461 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L461 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L462 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L462 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L463 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L463 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L464 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L464 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L465 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L465 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L466 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L466 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L467 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L467 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L468 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L468 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L469 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L469 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L470 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L470 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L471 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L471 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L472 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L472 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L473 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L473 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L474 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L474 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L475 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L475 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L476 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L476 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L477 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L477 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L478 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L478 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L479 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L479 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L480 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L480 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 481-504

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vslawsx_vvsvl : ClangBuiltin<"__builtin_ve_vl_vslawsx_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslawsx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vslawsx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslawsx_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vslawsx_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslawzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vslawzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslawzx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vslawzx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslawzx_vvsl : ClangBuiltin<"__builtin_ve_vl_vslawzx_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslawzx_vvsvl : ClangBuiltin<"__builtin_ve_vl_vslawzx_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslawzx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vslawzx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslawzx_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vslawzx_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsla_vvvl : ClangBuiltin<"__builtin_ve_vl_pvsla_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsla_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvsla_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsla_vvsl : ClangBuiltin<"__builtin_ve_vl_pvsla_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsla_vvsvl : ClangBuiltin<"__builtin_ve_vl_pvsla_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsla_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvsla_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsla_vvsMvl : ClangBuiltin<"__builtin_ve_vl_pvsla_vvsMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslal_vvvl : ClangBuiltin<"__builtin_ve_vl_vslal_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslal_vvvvl : ClangBuiltin<"__builtin_ve_vl_vslal_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslal_vvsl : ClangBuiltin<"__builtin_ve_vl_vslal_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslal_vvsvl : ClangBuiltin<"__builtin_ve_vl_vslal_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslal_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vslal_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vslal_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vslal_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrawsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vsrawsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrawsx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vsrawsx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrawsx_vvsl : ClangBuiltin<"__builtin_ve_vl_vsrawsx_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<i32>], [IntrNoMem]>;
````
- **L481 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L481 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L482 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L482 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L483 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L483 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L484 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L484 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L485 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L485 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L486 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L486 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L487 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L487 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L488 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L488 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L489 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L489 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L490 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L490 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L491 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L491 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L492 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L492 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L493 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L493 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L494 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L494 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L495 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L495 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L496 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L496 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L497 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L497 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L498 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L498 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L499 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L499 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L500 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L500 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L501 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L501 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L502 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L502 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L503 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L503 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L504 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L504 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 505-528

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vsrawsx_vvsvl : ClangBuiltin<"__builtin_ve_vl_vsrawsx_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrawsx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vsrawsx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrawsx_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vsrawsx_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrawzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vsrawzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrawzx_vvvvl : ClangBuiltin<"__builtin_ve_vl_vsrawzx_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrawzx_vvsl : ClangBuiltin<"__builtin_ve_vl_vsrawzx_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrawzx_vvsvl : ClangBuiltin<"__builtin_ve_vl_vsrawzx_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrawzx_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vsrawzx_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsrawzx_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vsrawzx_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsra_vvvl : ClangBuiltin<"__builtin_ve_vl_pvsra_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsra_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvsra_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsra_vvsl : ClangBuiltin<"__builtin_ve_vl_pvsra_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsra_vvsvl : ClangBuiltin<"__builtin_ve_vl_pvsra_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsra_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvsra_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvsra_vvsMvl : ClangBuiltin<"__builtin_ve_vl_pvsra_vvsMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsral_vvvl : ClangBuiltin<"__builtin_ve_vl_vsral_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsral_vvvvl : ClangBuiltin<"__builtin_ve_vl_vsral_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsral_vvsl : ClangBuiltin<"__builtin_ve_vl_vsral_vvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsral_vvsvl : ClangBuiltin<"__builtin_ve_vl_vsral_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsral_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vsral_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsral_vvsmvl : ClangBuiltin<"__builtin_ve_vl_vsral_vvsmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsfa_vvssl : ClangBuiltin<"__builtin_ve_vl_vsfa_vvssl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsfa_vvssvl : ClangBuiltin<"__builtin_ve_vl_vsfa_vvssvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsfa_vvssmvl : ClangBuiltin<"__builtin_ve_vl_vsfa_vvssmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L505 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L505 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L506 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L506 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L507 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L507 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L508 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L508 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L509 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L509 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L510 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L510 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L511 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L511 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L512 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L512 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L513 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L513 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L514 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L514 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L515 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L515 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L516 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L516 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L517 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L517 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L518 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L518 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L519 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L519 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L520 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L520 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L521 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L521 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L522 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L522 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L523 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L523 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L524 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L524 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L525 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L525 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L526 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L526 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L527 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L527 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L528 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L528 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 529-552

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfaddd_vvvl : ClangBuiltin<"__builtin_ve_vl_vfaddd_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfaddd_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfaddd_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfaddd_vsvl : ClangBuiltin<"__builtin_ve_vl_vfaddd_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfaddd_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfaddd_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfaddd_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfaddd_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfaddd_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfaddd_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfadds_vvvl : ClangBuiltin<"__builtin_ve_vl_vfadds_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfadds_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfadds_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfadds_vsvl : ClangBuiltin<"__builtin_ve_vl_vfadds_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfadds_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfadds_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfadds_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfadds_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfadds_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfadds_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfadd_vvvl : ClangBuiltin<"__builtin_ve_vl_pvfadd_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfadd_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvfadd_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfadd_vsvl : ClangBuiltin<"__builtin_ve_vl_pvfadd_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfadd_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvfadd_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfadd_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfadd_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfadd_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvfadd_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsubd_vvvl : ClangBuiltin<"__builtin_ve_vl_vfsubd_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsubd_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfsubd_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsubd_vsvl : ClangBuiltin<"__builtin_ve_vl_vfsubd_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsubd_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfsubd_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsubd_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfsubd_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsubd_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfsubd_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L529 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L529 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L530 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L530 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L531 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L531 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L532 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L532 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L533 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L533 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L534 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L534 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L535 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L535 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L536 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L536 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L537 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L537 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L538 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L538 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L539 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L539 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L540 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L540 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L541 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L541 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L542 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L542 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L543 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L543 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L544 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L544 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L545 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L545 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L546 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L546 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L547 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L547 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L548 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L548 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L549 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L549 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L550 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L550 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L551 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L551 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L552 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L552 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 553-576

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfsubs_vvvl : ClangBuiltin<"__builtin_ve_vl_vfsubs_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsubs_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfsubs_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsubs_vsvl : ClangBuiltin<"__builtin_ve_vl_vfsubs_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsubs_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfsubs_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsubs_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfsubs_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsubs_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfsubs_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfsub_vvvl : ClangBuiltin<"__builtin_ve_vl_pvfsub_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfsub_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvfsub_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfsub_vsvl : ClangBuiltin<"__builtin_ve_vl_pvfsub_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfsub_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvfsub_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfsub_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfsub_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfsub_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvfsub_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuld_vvvl : ClangBuiltin<"__builtin_ve_vl_vfmuld_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuld_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfmuld_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuld_vsvl : ClangBuiltin<"__builtin_ve_vl_vfmuld_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuld_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfmuld_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuld_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmuld_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuld_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfmuld_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuls_vvvl : ClangBuiltin<"__builtin_ve_vl_vfmuls_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuls_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfmuls_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuls_vsvl : ClangBuiltin<"__builtin_ve_vl_vfmuls_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuls_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfmuls_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuls_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmuls_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmuls_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfmuls_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L553 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L553 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L554 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L554 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L555 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L555 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L556 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L556 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L557 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L557 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L558 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L558 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L559 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L559 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L560 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L560 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L561 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L561 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L562 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L562 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L563 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L563 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L564 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L564 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L565 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L565 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L566 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L566 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L567 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L567 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L568 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L568 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L569 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L569 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L570 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L570 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L571 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L571 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L572 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L572 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L573 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L573 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L574 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L574 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L575 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L575 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L576 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L576 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 577-600

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvfmul_vvvl : ClangBuiltin<"__builtin_ve_vl_pvfmul_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmul_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvfmul_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmul_vsvl : ClangBuiltin<"__builtin_ve_vl_pvfmul_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmul_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvfmul_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmul_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmul_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmul_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmul_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivd_vvvl : ClangBuiltin<"__builtin_ve_vl_vfdivd_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivd_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfdivd_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivd_vsvl : ClangBuiltin<"__builtin_ve_vl_vfdivd_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivd_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfdivd_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivd_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfdivd_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivd_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfdivd_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivs_vvvl : ClangBuiltin<"__builtin_ve_vl_vfdivs_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivs_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfdivs_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivs_vsvl : ClangBuiltin<"__builtin_ve_vl_vfdivs_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivs_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfdivs_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivs_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfdivs_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfdivs_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfdivs_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsqrtd_vvl : ClangBuiltin<"__builtin_ve_vl_vfsqrtd_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsqrtd_vvvl : ClangBuiltin<"__builtin_ve_vl_vfsqrtd_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsqrts_vvl : ClangBuiltin<"__builtin_ve_vl_vfsqrts_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsqrts_vvvl : ClangBuiltin<"__builtin_ve_vl_vfsqrts_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfcmpd_vvvl : ClangBuiltin<"__builtin_ve_vl_vfcmpd_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfcmpd_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfcmpd_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L577 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L577 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L578 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L578 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L579 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L579 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L580 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L580 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L581 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L581 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L582 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L582 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L583 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L583 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L584 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L584 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L585 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L585 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L586 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L586 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L587 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L587 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L588 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L588 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L589 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L589 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L590 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L590 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L591 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L591 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L592 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L592 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L593 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L593 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L594 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L594 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L595 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L595 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L596 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L596 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L597 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L597 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L598 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L598 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L599 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L599 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L600 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L600 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 601-624

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfcmpd_vsvl : ClangBuiltin<"__builtin_ve_vl_vfcmpd_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfcmpd_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfcmpd_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfcmpd_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfcmpd_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfcmpd_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfcmpd_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfcmps_vvvl : ClangBuiltin<"__builtin_ve_vl_vfcmps_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfcmps_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfcmps_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfcmps_vsvl : ClangBuiltin<"__builtin_ve_vl_vfcmps_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfcmps_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfcmps_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfcmps_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfcmps_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfcmps_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfcmps_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfcmp_vvvl : ClangBuiltin<"__builtin_ve_vl_pvfcmp_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfcmp_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvfcmp_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfcmp_vsvl : ClangBuiltin<"__builtin_ve_vl_pvfcmp_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfcmp_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvfcmp_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfcmp_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfcmp_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfcmp_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvfcmp_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmaxd_vvvl : ClangBuiltin<"__builtin_ve_vl_vfmaxd_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmaxd_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfmaxd_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmaxd_vsvl : ClangBuiltin<"__builtin_ve_vl_vfmaxd_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmaxd_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfmaxd_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmaxd_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmaxd_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmaxd_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfmaxd_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmaxs_vvvl : ClangBuiltin<"__builtin_ve_vl_vfmaxs_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmaxs_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfmaxs_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L601 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L601 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L602 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L602 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L603 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L603 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L604 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L604 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L605 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L605 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L606 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L606 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L607 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L607 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L608 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L608 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L609 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L609 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L610 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L610 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L611 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L611 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L612 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L612 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L613 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L613 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L614 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L614 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L615 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L615 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L616 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L616 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L617 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L617 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L618 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L618 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L619 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L619 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L620 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L620 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L621 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L621 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L622 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L622 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L623 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L623 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L624 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L624 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 625-648

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfmaxs_vsvl : ClangBuiltin<"__builtin_ve_vl_vfmaxs_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmaxs_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfmaxs_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmaxs_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmaxs_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmaxs_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfmaxs_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmax_vvvl : ClangBuiltin<"__builtin_ve_vl_pvfmax_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmax_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvfmax_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmax_vsvl : ClangBuiltin<"__builtin_ve_vl_pvfmax_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmax_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvfmax_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmax_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmax_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmax_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmax_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmind_vvvl : ClangBuiltin<"__builtin_ve_vl_vfmind_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmind_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfmind_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmind_vsvl : ClangBuiltin<"__builtin_ve_vl_vfmind_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmind_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfmind_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmind_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmind_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmind_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfmind_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmins_vvvl : ClangBuiltin<"__builtin_ve_vl_vfmins_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmins_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfmins_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmins_vsvl : ClangBuiltin<"__builtin_ve_vl_vfmins_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmins_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfmins_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmins_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmins_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmins_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vfmins_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmin_vvvl : ClangBuiltin<"__builtin_ve_vl_pvfmin_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmin_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvfmin_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L625 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L625 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L626 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L626 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L627 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L627 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L628 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L628 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L629 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L629 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L630 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L630 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L631 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L631 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L632 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L632 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L633 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L633 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L634 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L634 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L635 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L635 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L636 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L636 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L637 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L637 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L638 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L638 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L639 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L639 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L640 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L640 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L641 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L641 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L642 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L642 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L643 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L643 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L644 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L644 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L645 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L645 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L646 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L646 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L647 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L647 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L648 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L648 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 649-672

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvfmin_vsvl : ClangBuiltin<"__builtin_ve_vl_pvfmin_vsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmin_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvfmin_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmin_vvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmin_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmin_vsvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmin_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmadd_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfmadd_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmadd_vvvvvl : ClangBuiltin<"__builtin_ve_vl_vfmadd_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmadd_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfmadd_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmadd_vsvvvl : ClangBuiltin<"__builtin_ve_vl_vfmadd_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmadd_vvsvl : ClangBuiltin<"__builtin_ve_vl_vfmadd_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmadd_vvsvvl : ClangBuiltin<"__builtin_ve_vl_vfmadd_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmadd_vvvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmadd_vvvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmadd_vsvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmadd_vsvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmadd_vvsvmvl : ClangBuiltin<"__builtin_ve_vl_vfmadd_vvsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmads_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfmads_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmads_vvvvvl : ClangBuiltin<"__builtin_ve_vl_vfmads_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmads_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfmads_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmads_vsvvvl : ClangBuiltin<"__builtin_ve_vl_vfmads_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmads_vvsvl : ClangBuiltin<"__builtin_ve_vl_vfmads_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmads_vvsvvl : ClangBuiltin<"__builtin_ve_vl_vfmads_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmads_vvvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmads_vvvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmads_vsvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmads_vsvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmads_vvsvmvl : ClangBuiltin<"__builtin_ve_vl_vfmads_vvsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmad_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvfmad_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmad_vvvvvl : ClangBuiltin<"__builtin_ve_vl_pvfmad_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L649 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L649 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L650 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L650 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L651 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L651 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L652 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L652 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L653 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L653 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L654 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L654 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L655 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L655 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L656 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L656 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L657 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L657 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L658 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L658 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L659 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L659 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L660 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L660 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L661 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L661 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L662 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L662 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L663 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L663 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L664 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L664 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L665 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L665 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L666 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L666 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L667 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L667 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L668 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L668 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L669 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L669 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L670 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L670 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L671 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L671 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L672 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L672 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 673-696

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvfmad_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvfmad_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmad_vsvvvl : ClangBuiltin<"__builtin_ve_vl_pvfmad_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmad_vvsvl : ClangBuiltin<"__builtin_ve_vl_pvfmad_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmad_vvsvvl : ClangBuiltin<"__builtin_ve_vl_pvfmad_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmad_vvvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmad_vvvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmad_vsvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmad_vsvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmad_vvsvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmad_vvsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbd_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfmsbd_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbd_vvvvvl : ClangBuiltin<"__builtin_ve_vl_vfmsbd_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbd_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfmsbd_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbd_vsvvvl : ClangBuiltin<"__builtin_ve_vl_vfmsbd_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbd_vvsvl : ClangBuiltin<"__builtin_ve_vl_vfmsbd_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbd_vvsvvl : ClangBuiltin<"__builtin_ve_vl_vfmsbd_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbd_vvvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmsbd_vvvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbd_vsvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmsbd_vsvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbd_vvsvmvl : ClangBuiltin<"__builtin_ve_vl_vfmsbd_vvsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbs_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfmsbs_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbs_vvvvvl : ClangBuiltin<"__builtin_ve_vl_vfmsbs_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbs_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfmsbs_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbs_vsvvvl : ClangBuiltin<"__builtin_ve_vl_vfmsbs_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbs_vvsvl : ClangBuiltin<"__builtin_ve_vl_vfmsbs_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbs_vvsvvl : ClangBuiltin<"__builtin_ve_vl_vfmsbs_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbs_vvvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmsbs_vvvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmsbs_vsvvmvl : ClangBuiltin<"__builtin_ve_vl_vfmsbs_vsvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L673 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L673 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L674 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L674 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L675 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L675 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L676 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L676 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L677 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L677 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L678 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L678 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L679 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L679 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L680 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L680 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L681 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L681 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L682 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L682 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L683 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L683 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L684 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L684 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L685 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L685 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L686 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L686 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L687 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L687 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L688 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L688 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L689 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L689 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L690 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L690 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L691 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L691 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L692 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L692 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L693 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L693 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L694 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L694 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L695 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L695 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L696 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L696 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 697-720

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfmsbs_vvsvmvl : ClangBuiltin<"__builtin_ve_vl_vfmsbs_vvsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmsb_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvfmsb_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmsb_vvvvvl : ClangBuiltin<"__builtin_ve_vl_pvfmsb_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmsb_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvfmsb_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmsb_vsvvvl : ClangBuiltin<"__builtin_ve_vl_pvfmsb_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmsb_vvsvl : ClangBuiltin<"__builtin_ve_vl_pvfmsb_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmsb_vvsvvl : ClangBuiltin<"__builtin_ve_vl_pvfmsb_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmsb_vvvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmsb_vvvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmsb_vsvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmsb_vsvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmsb_vvsvMvl : ClangBuiltin<"__builtin_ve_vl_pvfmsb_vvsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmadd_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmadd_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmadd_vvvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmadd_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmadd_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfnmadd_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmadd_vsvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmadd_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmadd_vvsvl : ClangBuiltin<"__builtin_ve_vl_vfnmadd_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmadd_vvsvvl : ClangBuiltin<"__builtin_ve_vl_vfnmadd_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmadd_vvvvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmadd_vvvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmadd_vsvvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmadd_vsvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmadd_vvsvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmadd_vvsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmads_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmads_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmads_vvvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmads_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmads_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfnmads_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmads_vsvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmads_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmads_vvsvl : ClangBuiltin<"__builtin_ve_vl_vfnmads_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L697 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L697 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L698 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L698 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L699 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L699 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L700 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L700 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L701 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L701 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L702 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L702 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L703 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L703 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L704 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L704 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L705 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L705 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L706 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L706 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L707 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L707 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L708 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L708 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L709 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L709 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L710 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L710 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L711 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L711 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L712 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L712 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L713 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L713 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L714 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L714 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L715 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L715 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L716 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L716 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L717 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L717 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L718 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L718 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L719 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L719 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L720 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L720 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 721-744

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfnmads_vvsvvl : ClangBuiltin<"__builtin_ve_vl_vfnmads_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmads_vvvvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmads_vvvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmads_vsvvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmads_vsvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmads_vvsvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmads_vvsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmad_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvfnmad_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmad_vvvvvl : ClangBuiltin<"__builtin_ve_vl_pvfnmad_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmad_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvfnmad_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmad_vsvvvl : ClangBuiltin<"__builtin_ve_vl_pvfnmad_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmad_vvsvl : ClangBuiltin<"__builtin_ve_vl_pvfnmad_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmad_vvsvvl : ClangBuiltin<"__builtin_ve_vl_pvfnmad_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmad_vvvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfnmad_vvvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmad_vsvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfnmad_vsvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmad_vvsvMvl : ClangBuiltin<"__builtin_ve_vl_pvfnmad_vvsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbd_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbd_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbd_vvvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbd_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbd_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbd_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbd_vsvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbd_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbd_vvsvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbd_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbd_vvsvvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbd_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbd_vvvvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbd_vvvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbd_vsvvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbd_vsvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbd_vvsvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbd_vvsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbs_vvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbs_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbs_vvvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbs_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L721 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L721 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L722 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L722 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L723 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L723 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L724 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L724 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L725 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L725 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L726 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L726 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L727 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L727 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L728 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L728 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L729 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L729 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L730 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L730 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L731 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L731 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L732 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L732 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L733 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L733 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L734 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L734 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L735 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L735 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L736 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L736 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L737 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L737 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L738 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L738 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L739 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L739 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L740 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L740 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L741 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L741 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L742 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L742 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L743 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L743 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L744 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L744 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 745-768

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbs_vsvvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbs_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbs_vsvvvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbs_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbs_vvsvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbs_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbs_vvsvvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbs_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbs_vvvvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbs_vvvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbs_vsvvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbs_vsvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfnmsbs_vvsvmvl : ClangBuiltin<"__builtin_ve_vl_vfnmsbs_vvsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<f32>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmsb_vvvvl : ClangBuiltin<"__builtin_ve_vl_pvfnmsb_vvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmsb_vvvvvl : ClangBuiltin<"__builtin_ve_vl_pvfnmsb_vvvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmsb_vsvvl : ClangBuiltin<"__builtin_ve_vl_pvfnmsb_vsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmsb_vsvvvl : ClangBuiltin<"__builtin_ve_vl_pvfnmsb_vsvvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmsb_vvsvl : ClangBuiltin<"__builtin_ve_vl_pvfnmsb_vvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmsb_vvsvvl : ClangBuiltin<"__builtin_ve_vl_pvfnmsb_vvsvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmsb_vvvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfnmsb_vvvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmsb_vsvvMvl : ClangBuiltin<"__builtin_ve_vl_pvfnmsb_vsvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfnmsb_vvsvMvl : ClangBuiltin<"__builtin_ve_vl_pvfnmsb_vvsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrcpd_vvl : ClangBuiltin<"__builtin_ve_vl_vrcpd_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrcpd_vvvl : ClangBuiltin<"__builtin_ve_vl_vrcpd_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrcps_vvl : ClangBuiltin<"__builtin_ve_vl_vrcps_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrcps_vvvl : ClangBuiltin<"__builtin_ve_vl_vrcps_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvrcp_vvl : ClangBuiltin<"__builtin_ve_vl_pvrcp_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvrcp_vvvl : ClangBuiltin<"__builtin_ve_vl_pvrcp_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrsqrtd_vvl : ClangBuiltin<"__builtin_ve_vl_vrsqrtd_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrsqrtd_vvvl : ClangBuiltin<"__builtin_ve_vl_vrsqrtd_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L745 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L745 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L746 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L746 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L747 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L747 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L748 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L748 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L749 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L749 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L750 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L750 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L751 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L751 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L752 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L752 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L753 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L753 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L754 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L754 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L755 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L755 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L756 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L756 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L757 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L757 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L758 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L758 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L759 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L759 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L760 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L760 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L761 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L761 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L762 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L762 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L763 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L763 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L764 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L764 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L765 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L765 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L766 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L766 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L767 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L767 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L768 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L768 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 769-792

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vrsqrts_vvl : ClangBuiltin<"__builtin_ve_vl_vrsqrts_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrsqrts_vvvl : ClangBuiltin<"__builtin_ve_vl_vrsqrts_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvrsqrt_vvl : ClangBuiltin<"__builtin_ve_vl_pvrsqrt_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvrsqrt_vvvl : ClangBuiltin<"__builtin_ve_vl_pvrsqrt_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrsqrtdnex_vvl : ClangBuiltin<"__builtin_ve_vl_vrsqrtdnex_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrsqrtdnex_vvvl : ClangBuiltin<"__builtin_ve_vl_vrsqrtdnex_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrsqrtsnex_vvl : ClangBuiltin<"__builtin_ve_vl_vrsqrtsnex_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrsqrtsnex_vvvl : ClangBuiltin<"__builtin_ve_vl_vrsqrtsnex_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvrsqrtnex_vvl : ClangBuiltin<"__builtin_ve_vl_pvrsqrtnex_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvrsqrtnex_vvvl : ClangBuiltin<"__builtin_ve_vl_pvrsqrtnex_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdsx_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdsx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdsx_vvmvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdsx_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdsxrz_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdsxrz_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdsxrz_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdsxrz_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdsxrz_vvmvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdsxrz_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdzx_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdzx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdzx_vvmvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdzx_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdzxrz_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdzxrz_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdzxrz_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdzxrz_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwdzxrz_vvmvl : ClangBuiltin<"__builtin_ve_vl_vcvtwdzxrz_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwssx_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtwssx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwssx_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtwssx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L769 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L769 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L770 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L770 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L771 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L771 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L772 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L772 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L773 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L773 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L774 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L774 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L775 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L775 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L776 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L776 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L777 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L777 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L778 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L778 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L779 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L779 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L780 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L780 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L781 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L781 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L782 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L782 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L783 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L783 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L784 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L784 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L785 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L785 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L786 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L786 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L787 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L787 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L788 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L788 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L789 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L789 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L790 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L790 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L791 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L791 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L792 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L792 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 793-816

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vcvtwssx_vvmvl : ClangBuiltin<"__builtin_ve_vl_vcvtwssx_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwssxrz_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtwssxrz_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwssxrz_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtwssxrz_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwssxrz_vvmvl : ClangBuiltin<"__builtin_ve_vl_vcvtwssxrz_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwszx_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtwszx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwszx_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtwszx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwszx_vvmvl : ClangBuiltin<"__builtin_ve_vl_vcvtwszx_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwszxrz_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtwszxrz_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwszxrz_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtwszxrz_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtwszxrz_vvmvl : ClangBuiltin<"__builtin_ve_vl_vcvtwszxrz_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcvtws_vvl : ClangBuiltin<"__builtin_ve_vl_pvcvtws_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcvtws_vvvl : ClangBuiltin<"__builtin_ve_vl_pvcvtws_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcvtws_vvMvl : ClangBuiltin<"__builtin_ve_vl_pvcvtws_vvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcvtwsrz_vvl : ClangBuiltin<"__builtin_ve_vl_pvcvtwsrz_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcvtwsrz_vvvl : ClangBuiltin<"__builtin_ve_vl_pvcvtwsrz_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcvtwsrz_vvMvl : ClangBuiltin<"__builtin_ve_vl_pvcvtwsrz_vvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtld_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtld_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtld_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtld_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtld_vvmvl : ClangBuiltin<"__builtin_ve_vl_vcvtld_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtldrz_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtldrz_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtldrz_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtldrz_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtldrz_vvmvl : ClangBuiltin<"__builtin_ve_vl_vcvtldrz_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtdw_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtdw_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtdw_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtdw_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L793 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L793 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L794 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L794 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L795 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L795 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L796 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L796 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L797 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L797 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L798 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L798 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L799 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L799 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L800 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L800 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L801 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L801 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L802 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L802 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L803 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L803 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L804 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L804 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L805 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L805 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L806 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L806 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L807 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L807 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L808 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L808 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L809 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L809 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L810 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L810 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L811 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L811 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L812 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L812 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L813 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L813 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L814 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L814 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L815 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L815 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L816 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L816 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 817-840

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vcvtsw_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtsw_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtsw_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtsw_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcvtsw_vvl : ClangBuiltin<"__builtin_ve_vl_pvcvtsw_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvcvtsw_vvvl : ClangBuiltin<"__builtin_ve_vl_pvcvtsw_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtdl_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtdl_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtdl_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtdl_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtds_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtds_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtds_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtds_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtsd_vvl : ClangBuiltin<"__builtin_ve_vl_vcvtsd_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcvtsd_vvvl : ClangBuiltin<"__builtin_ve_vl_vcvtsd_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmrg_vvvml : ClangBuiltin<"__builtin_ve_vl_vmrg_vvvml">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmrg_vvvmvl : ClangBuiltin<"__builtin_ve_vl_vmrg_vvvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmrg_vsvml : ClangBuiltin<"__builtin_ve_vl_vmrg_vsvml">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmrg_vsvmvl : ClangBuiltin<"__builtin_ve_vl_vmrg_vsvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i64>, LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmrgw_vvvMl : ClangBuiltin<"__builtin_ve_vl_vmrgw_vvvMl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmrgw_vvvMvl : ClangBuiltin<"__builtin_ve_vl_vmrgw_vvvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmrgw_vsvMl : ClangBuiltin<"__builtin_ve_vl_vmrgw_vsvMl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vmrgw_vsvMvl : ClangBuiltin<"__builtin_ve_vl_vmrgw_vsvMvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<i32>, LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vshf_vvvsl : ClangBuiltin<"__builtin_ve_vl_vshf_vvvsl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vshf_vvvsvl : ClangBuiltin<"__builtin_ve_vl_vshf_vvvsvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vcp_vvmvl : ClangBuiltin<"__builtin_ve_vl_vcp_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vex_vvmvl : ClangBuiltin<"__builtin_ve_vl_vex_vvmvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklat_ml : ClangBuiltin<"__builtin_ve_vl_vfmklat_ml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklaf_ml : ClangBuiltin<"__builtin_ve_vl_vfmklaf_ml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<i32>], [IntrNoMem]>;
````
- **L817 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L817 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L818 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L818 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L819 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L819 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L820 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L820 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L821 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L821 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L822 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L822 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L823 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L823 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L824 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L824 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L825 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L825 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L826 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L826 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L827 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L827 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L828 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L828 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L829 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L829 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L830 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L830 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L831 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L831 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L832 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L832 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L833 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L833 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L834 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L834 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L835 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L835 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L836 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L836 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L837 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L837 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L838 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L838 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L839 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L839 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L840 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L840 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 841-864

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvfmkat_Ml : ClangBuiltin<"__builtin_ve_vl_pvfmkat_Ml">, Intrinsic<[LLVMType<v512i1>], [LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkaf_Ml : ClangBuiltin<"__builtin_ve_vl_pvfmkaf_Ml">, Intrinsic<[LLVMType<v512i1>], [LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklgt_mvl : ClangBuiltin<"__builtin_ve_vl_vfmklgt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklgt_mvml : ClangBuiltin<"__builtin_ve_vl_vfmklgt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkllt_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkllt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkllt_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkllt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklne_mvl : ClangBuiltin<"__builtin_ve_vl_vfmklne_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklne_mvml : ClangBuiltin<"__builtin_ve_vl_vfmklne_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkleq_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkleq_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkleq_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkleq_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklge_mvl : ClangBuiltin<"__builtin_ve_vl_vfmklge_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklge_mvml : ClangBuiltin<"__builtin_ve_vl_vfmklge_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklle_mvl : ClangBuiltin<"__builtin_ve_vl_vfmklle_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklle_mvml : ClangBuiltin<"__builtin_ve_vl_vfmklle_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklnum_mvl : ClangBuiltin<"__builtin_ve_vl_vfmklnum_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklnum_mvml : ClangBuiltin<"__builtin_ve_vl_vfmklnum_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmklnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmklnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklgtnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmklgtnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklgtnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmklgtnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklltnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmklltnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklltnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmklltnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklnenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmklnenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklnenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmklnenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
````
- **L841 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L841 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L842 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L842 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L843 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L843 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L844 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L844 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L845 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L845 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L846 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L846 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L847 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L847 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L848 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L848 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L849 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L849 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L850 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L850 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L851 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L851 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L852 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L852 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L853 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L853 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L854 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L854 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L855 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L855 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L856 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L856 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L857 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L857 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L858 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L858 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L859 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L859 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L860 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L860 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L861 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L861 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L862 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L862 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L863 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L863 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L864 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L864 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 865-888

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfmkleqnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkleqnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkleqnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkleqnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklgenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmklgenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmklgenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmklgenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkllenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkllenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkllenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkllenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwgt_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwgt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwgt_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwgt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwlt_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwlt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwlt_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwlt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwne_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwne_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwne_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwne_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkweq_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkweq_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkweq_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkweq_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwge_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwge_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwge_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwge_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwle_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwle_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwle_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwle_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwnum_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwnum_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwnum_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwnum_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwgtnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwgtnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwgtnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwgtnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
````
- **L865 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L865 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L866 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L866 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L867 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L867 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L868 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L868 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L869 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L869 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L870 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L870 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L871 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L871 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L872 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L872 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L873 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L873 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L874 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L874 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L875 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L875 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L876 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L876 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L877 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L877 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L878 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L878 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L879 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L879 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L880 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L880 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L881 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L881 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L882 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L882 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L883 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L883 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L884 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L884 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L885 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L885 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L886 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L886 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L887 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L887 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L888 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L888 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 889-912

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfmkwltnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwltnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwltnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwltnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwnenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwnenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwnenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwnenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkweqnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkweqnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkweqnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkweqnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwgenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwgenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwgenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwgenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwlenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkwlenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkwlenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkwlenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlogt_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlogt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupgt_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwupgt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlogt_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwlogt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupgt_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwupgt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlolt_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlolt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwuplt_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwuplt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlolt_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwlolt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwuplt_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwuplt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlone_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlone_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupne_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwupne_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlone_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwlone_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupne_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwupne_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwloeq_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwloeq_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupeq_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwupeq_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L889 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L889 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L890 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L890 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L891 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L891 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L892 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L892 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L893 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L893 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L894 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L894 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L895 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L895 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L896 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L896 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L897 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L897 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L898 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L898 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L899 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L899 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L900 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L900 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L901 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L901 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L902 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L902 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L903 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L903 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L904 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L904 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L905 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L905 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L906 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L906 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L907 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L907 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L908 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L908 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L909 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L909 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L910 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L910 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L911 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L911 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L912 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L912 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 913-936

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwloeq_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwloeq_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupeq_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwupeq_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwloge_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwloge_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupge_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwupge_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwloge_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwloge_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupge_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwupge_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlole_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlole_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwuple_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwuple_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlole_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwlole_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwuple_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwuple_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlonum_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlonum_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupnum_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwupnum_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlonum_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwlonum_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupnum_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwupnum_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlonan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlonan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwupnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlonan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwlonan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwupnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlogtnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlogtnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupgtnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwupgtnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlogtnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwlogtnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupgtnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwupgtnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwloltnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwloltnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupltnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwupltnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L913 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L913 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L914 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L914 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L915 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L915 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L916 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L916 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L917 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L917 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L918 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L918 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L919 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L919 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L920 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L920 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L921 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L921 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L922 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L922 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L923 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L923 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L924 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L924 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L925 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L925 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L926 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L926 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L927 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L927 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L928 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L928 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L929 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L929 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L930 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L930 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L931 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L931 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L932 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L932 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L933 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L933 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L934 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L934 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L935 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L935 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L936 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L936 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 937-960

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwloltnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwloltnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupltnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwupltnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlonenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlonenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupnenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwupnenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlonenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwlonenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupnenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwupnenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwloeqnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwloeqnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupeqnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwupeqnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwloeqnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwloeqnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupeqnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwupeqnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlogenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlogenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupgenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwupgenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlogenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwlogenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwupgenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwupgenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlolenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlolenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwuplenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwuplenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlolenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwlolenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwuplenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkwuplenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwgt_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwgt_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwgt_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwgt_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlt_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlt_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlt_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlt_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwne_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwne_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwne_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwne_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
````
- **L937 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L937 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L938 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L938 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L939 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L939 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L940 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L940 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L941 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L941 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L942 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L942 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L943 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L943 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L944 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L944 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L945 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L945 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L946 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L946 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L947 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L947 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L948 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L948 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L949 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L949 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L950 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L950 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L951 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L951 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L952 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L952 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L953 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L953 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L954 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L954 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L955 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L955 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L956 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L956 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L957 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L957 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L958 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L958 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L959 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L959 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L960 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L960 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 961-984

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvfmkweq_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkweq_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkweq_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkweq_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwge_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwge_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwge_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwge_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwle_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwle_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwle_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwle_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwnum_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwnum_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwnum_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwnum_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwnan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwnan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwnan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwnan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwgtnan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwgtnan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwgtnan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwgtnan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwltnan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwltnan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwltnan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwltnan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwnenan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwnenan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwnenan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwnenan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkweqnan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkweqnan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkweqnan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkweqnan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwgenan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwgenan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwgenan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwgenan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlenan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlenan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkwlenan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkwlenan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdgt_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdgt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdgt_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdgt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
````
- **L961 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L961 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L962 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L962 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L963 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L963 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L964 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L964 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L965 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L965 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L966 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L966 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L967 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L967 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L968 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L968 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L969 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L969 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L970 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L970 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L971 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L971 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L972 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L972 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L973 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L973 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L974 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L974 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L975 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L975 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L976 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L976 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L977 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L977 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L978 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L978 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L979 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L979 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L980 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L980 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L981 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L981 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L982 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L982 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L983 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L983 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L984 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L984 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 985-1008

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfmkdlt_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdlt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdlt_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdlt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdne_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdne_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdne_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdne_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdeq_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdeq_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdeq_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdeq_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdge_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdge_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdge_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdge_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdle_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdle_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdle_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdle_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdnum_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdnum_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdnum_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdnum_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdgtnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdgtnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdgtnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdgtnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdltnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdltnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdltnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdltnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdnenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdnenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdnenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdnenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdeqnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdeqnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdeqnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdeqnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdgenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdgenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdgenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdgenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
````
- **L985 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L985 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L986 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L986 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L987 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L987 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L988 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L988 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L989 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L989 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L990 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L990 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L991 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L991 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L992 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L992 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L993 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L993 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L994 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L994 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L995 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L995 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L996 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L996 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L997 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L997 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L998 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L998 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L999 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L999 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1000 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1000 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1001 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1001 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1002 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1002 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1003 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1003 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1004 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1004 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1005 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1005 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1006 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1006 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1007 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1007 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1008 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1008 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1009-1032

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfmkdlenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkdlenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkdlenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkdlenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksgt_mvl : ClangBuiltin<"__builtin_ve_vl_vfmksgt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksgt_mvml : ClangBuiltin<"__builtin_ve_vl_vfmksgt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkslt_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkslt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkslt_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkslt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksne_mvl : ClangBuiltin<"__builtin_ve_vl_vfmksne_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksne_mvml : ClangBuiltin<"__builtin_ve_vl_vfmksne_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkseq_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkseq_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkseq_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkseq_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksge_mvl : ClangBuiltin<"__builtin_ve_vl_vfmksge_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksge_mvml : ClangBuiltin<"__builtin_ve_vl_vfmksge_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksle_mvl : ClangBuiltin<"__builtin_ve_vl_vfmksle_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksle_mvml : ClangBuiltin<"__builtin_ve_vl_vfmksle_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksnum_mvl : ClangBuiltin<"__builtin_ve_vl_vfmksnum_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksnum_mvml : ClangBuiltin<"__builtin_ve_vl_vfmksnum_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmksnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmksnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksgtnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmksgtnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksgtnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmksgtnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksltnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmksltnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksltnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmksltnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksnenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmksnenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksnenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmksnenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
````
- **L1009 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1009 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1010 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1010 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1011 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1011 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1012 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1012 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1013 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1013 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1014 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1014 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1015 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1015 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1016 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1016 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1017 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1017 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1018 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1018 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1019 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1019 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1020 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1020 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1021 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1021 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1022 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1022 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1023 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1023 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1024 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1024 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1025 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1025 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1026 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1026 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1027 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1027 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1028 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1028 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1029 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1029 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1030 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1030 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1031 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1031 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1032 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1032 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1033-1056

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfmkseqnan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkseqnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkseqnan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkseqnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksgenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmksgenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmksgenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmksgenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkslenan_mvl : ClangBuiltin<"__builtin_ve_vl_vfmkslenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfmkslenan_mvml : ClangBuiltin<"__builtin_ve_vl_vfmkslenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslogt_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslogt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupgt_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksupgt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslogt_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkslogt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupgt_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksupgt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslolt_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslolt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksuplt_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksuplt_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslolt_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkslolt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksuplt_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksuplt_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslone_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslone_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupne_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksupne_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslone_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkslone_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupne_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksupne_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksloeq_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksloeq_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupeq_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksupeq_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksloeq_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksloeq_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupeq_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksupeq_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksloge_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksloge_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupge_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksupge_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L1033 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1033 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1034 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1034 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1035 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1035 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1036 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1036 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1037 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1037 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1038 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1038 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1039 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1039 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1040 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1040 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1041 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1041 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1042 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1042 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1043 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1043 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1044 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1044 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1045 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1045 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1046 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1046 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1047 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1047 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1048 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1048 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1049 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1049 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1050 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1050 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1051 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1051 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1052 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1052 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1053 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1053 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1054 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1054 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1055 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1055 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1056 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1056 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1057-1080

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvfmksloge_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksloge_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupge_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksupge_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslole_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslole_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksuple_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksuple_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslole_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkslole_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksuple_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksuple_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslonum_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslonum_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupnum_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksupnum_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslonum_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkslonum_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupnum_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksupnum_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslonan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslonan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksupnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslonan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkslonan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksupnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslogtnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslogtnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupgtnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksupgtnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslogtnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkslogtnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupgtnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksupgtnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksloltnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksloltnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupltnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksupltnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksloltnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksloltnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupltnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksupltnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslonenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslonenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupnenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksupnenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L1057 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1057 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1058 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1058 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1059 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1059 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1060 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1060 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1061 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1061 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1062 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1062 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1063 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1063 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1064 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1064 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1065 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1065 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1066 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1066 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1067 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1067 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1068 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1068 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1069 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1069 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1070 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1070 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1071 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1071 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1072 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1072 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1073 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1073 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1074 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1074 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1075 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1075 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1076 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1076 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1077 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1077 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1078 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1078 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1079 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1079 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1080 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1080 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1081-1104

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslonenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkslonenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupnenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksupnenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksloeqnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksloeqnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupeqnan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksupeqnan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksloeqnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksloeqnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupeqnan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksupeqnan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslogenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslogenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupgenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksupgenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslogenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkslogenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksupgenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksupgenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslolenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslolenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksuplenan_mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksuplenan_mvl">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslolenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmkslolenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksuplenan_mvml : ClangBuiltin<"__builtin_ve_vl_pvfmksuplenan_mvml">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksgt_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksgt_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksgt_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmksgt_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslt_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslt_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslt_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkslt_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksne_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksne_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksne_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmksne_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkseq_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkseq_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkseq_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkseq_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksge_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksge_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksge_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmksge_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
````
- **L1081 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1081 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1082 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1082 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1083 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1083 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1084 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1084 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1085 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1085 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1086 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1086 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1087 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1087 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1088 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1088 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1089 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1089 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1090 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1090 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1091 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1091 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1092 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1092 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1093 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1093 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1094 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1094 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1095 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1095 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1096 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1096 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1097 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1097 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1098 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1098 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1099 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1099 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1100 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1100 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1101 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1101 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1102 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1102 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1103 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1103 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1104 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1104 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1105-1128

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_pvfmksle_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksle_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksle_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmksle_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksnum_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksnum_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksnum_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmksnum_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksnan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksnan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksnan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmksnan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksgtnan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksgtnan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksgtnan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmksgtnan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksltnan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksltnan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksltnan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmksltnan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksnenan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksnenan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksnenan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmksnenan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkseqnan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkseqnan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkseqnan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkseqnan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksgenan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmksgenan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmksgenan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmksgenan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslenan_Mvl : ClangBuiltin<"__builtin_ve_vl_pvfmkslenan_Mvl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pvfmkslenan_MvMl : ClangBuiltin<"__builtin_ve_vl_pvfmkslenan_MvMl">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v256f64>, LLVMType<v512i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsumwsx_vvl : ClangBuiltin<"__builtin_ve_vl_vsumwsx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsumwsx_vvml : ClangBuiltin<"__builtin_ve_vl_vsumwsx_vvml">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsumwzx_vvl : ClangBuiltin<"__builtin_ve_vl_vsumwzx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsumwzx_vvml : ClangBuiltin<"__builtin_ve_vl_vsumwzx_vvml">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsuml_vvl : ClangBuiltin<"__builtin_ve_vl_vsuml_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsuml_vvml : ClangBuiltin<"__builtin_ve_vl_vsuml_vvml">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
````
- **L1105 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1105 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1106 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1106 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1107 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1107 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1108 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1108 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1109 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1109 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1110 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1110 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1111 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1111 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1112 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1112 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1113 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1113 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1114 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1114 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1115 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1115 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1116 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1116 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1117 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1117 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1118 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1118 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1119 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1119 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1120 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1120 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1121 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1121 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1122 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1122 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1123 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1123 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1124 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1124 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1125 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1125 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1126 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1126 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1127 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1127 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1128 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1128 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1129-1152

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vfsumd_vvl : ClangBuiltin<"__builtin_ve_vl_vfsumd_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsumd_vvml : ClangBuiltin<"__builtin_ve_vl_vfsumd_vvml">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsums_vvl : ClangBuiltin<"__builtin_ve_vl_vfsums_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfsums_vvml : ClangBuiltin<"__builtin_ve_vl_vfsums_vvml">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxswfstsx_vvl : ClangBuiltin<"__builtin_ve_vl_vrmaxswfstsx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxswfstsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vrmaxswfstsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxswlstsx_vvl : ClangBuiltin<"__builtin_ve_vl_vrmaxswlstsx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxswlstsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vrmaxswlstsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxswfstzx_vvl : ClangBuiltin<"__builtin_ve_vl_vrmaxswfstzx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxswfstzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vrmaxswfstzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxswlstzx_vvl : ClangBuiltin<"__builtin_ve_vl_vrmaxswlstzx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxswlstzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vrmaxswlstzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrminswfstsx_vvl : ClangBuiltin<"__builtin_ve_vl_vrminswfstsx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrminswfstsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vrminswfstsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrminswlstsx_vvl : ClangBuiltin<"__builtin_ve_vl_vrminswlstsx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrminswlstsx_vvvl : ClangBuiltin<"__builtin_ve_vl_vrminswlstsx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrminswfstzx_vvl : ClangBuiltin<"__builtin_ve_vl_vrminswfstzx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrminswfstzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vrminswfstzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrminswlstzx_vvl : ClangBuiltin<"__builtin_ve_vl_vrminswlstzx_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrminswlstzx_vvvl : ClangBuiltin<"__builtin_ve_vl_vrminswlstzx_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxslfst_vvl : ClangBuiltin<"__builtin_ve_vl_vrmaxslfst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxslfst_vvvl : ClangBuiltin<"__builtin_ve_vl_vrmaxslfst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxsllst_vvl : ClangBuiltin<"__builtin_ve_vl_vrmaxsllst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrmaxsllst_vvvl : ClangBuiltin<"__builtin_ve_vl_vrmaxsllst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
````
- **L1129 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1129 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1130 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1130 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1131 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1131 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1132 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1132 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1133 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1133 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1134 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1134 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1135 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1135 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1136 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1136 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1137 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1137 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1138 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1138 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1139 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1139 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1140 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1140 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1141 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1141 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1142 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1142 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1143 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1143 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1144 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1144 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1145 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1145 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1146 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1146 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1147 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1147 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1148 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1148 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1149 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1149 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1150 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1150 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1151 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1151 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1152 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1152 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1153-1176

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vrminslfst_vvl : ClangBuiltin<"__builtin_ve_vl_vrminslfst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrminslfst_vvvl : ClangBuiltin<"__builtin_ve_vl_vrminslfst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrminsllst_vvl : ClangBuiltin<"__builtin_ve_vl_vrminsllst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrminsllst_vvvl : ClangBuiltin<"__builtin_ve_vl_vrminsllst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmaxdfst_vvl : ClangBuiltin<"__builtin_ve_vl_vfrmaxdfst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmaxdfst_vvvl : ClangBuiltin<"__builtin_ve_vl_vfrmaxdfst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmaxdlst_vvl : ClangBuiltin<"__builtin_ve_vl_vfrmaxdlst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmaxdlst_vvvl : ClangBuiltin<"__builtin_ve_vl_vfrmaxdlst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmaxsfst_vvl : ClangBuiltin<"__builtin_ve_vl_vfrmaxsfst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmaxsfst_vvvl : ClangBuiltin<"__builtin_ve_vl_vfrmaxsfst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmaxslst_vvl : ClangBuiltin<"__builtin_ve_vl_vfrmaxslst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmaxslst_vvvl : ClangBuiltin<"__builtin_ve_vl_vfrmaxslst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmindfst_vvl : ClangBuiltin<"__builtin_ve_vl_vfrmindfst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmindfst_vvvl : ClangBuiltin<"__builtin_ve_vl_vfrmindfst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmindlst_vvl : ClangBuiltin<"__builtin_ve_vl_vfrmindlst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrmindlst_vvvl : ClangBuiltin<"__builtin_ve_vl_vfrmindlst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrminsfst_vvl : ClangBuiltin<"__builtin_ve_vl_vfrminsfst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrminsfst_vvvl : ClangBuiltin<"__builtin_ve_vl_vfrminsfst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrminslst_vvl : ClangBuiltin<"__builtin_ve_vl_vfrminslst_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vfrminslst_vvvl : ClangBuiltin<"__builtin_ve_vl_vfrminslst_vvvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrand_vvl : ClangBuiltin<"__builtin_ve_vl_vrand_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrand_vvml : ClangBuiltin<"__builtin_ve_vl_vrand_vvml">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vror_vvl : ClangBuiltin<"__builtin_ve_vl_vror_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vror_vvml : ClangBuiltin<"__builtin_ve_vl_vror_vvml">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
````
- **L1153 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1153 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1154 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1154 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1155 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1155 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1156 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1156 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1157 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1157 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1158 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1158 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1159 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1159 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1160 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1160 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1161 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1161 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1162 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1162 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1163 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1163 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1164 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1164 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1165 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1165 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1166 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1166 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1167 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1167 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1168 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1168 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1169 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1169 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1170 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1170 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1171 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1171 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1172 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1172 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1173 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1173 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1174 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1174 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1175 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1175 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1176 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1176 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1177-1200

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vrxor_vvl : ClangBuiltin<"__builtin_ve_vl_vrxor_vvl">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vrxor_vvml : ClangBuiltin<"__builtin_ve_vl_vrxor_vvml">, Intrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgt_vvssl : ClangBuiltin<"__builtin_ve_vl_vgt_vvssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgt_vvssvl : ClangBuiltin<"__builtin_ve_vl_vgt_vvssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgt_vvssml : ClangBuiltin<"__builtin_ve_vl_vgt_vvssml">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgt_vvssmvl : ClangBuiltin<"__builtin_ve_vl_vgt_vvssmvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtnc_vvssl : ClangBuiltin<"__builtin_ve_vl_vgtnc_vvssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtnc_vvssvl : ClangBuiltin<"__builtin_ve_vl_vgtnc_vvssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtnc_vvssml : ClangBuiltin<"__builtin_ve_vl_vgtnc_vvssml">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtnc_vvssmvl : ClangBuiltin<"__builtin_ve_vl_vgtnc_vvssmvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtu_vvssl : ClangBuiltin<"__builtin_ve_vl_vgtu_vvssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtu_vvssvl : ClangBuiltin<"__builtin_ve_vl_vgtu_vvssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtu_vvssml : ClangBuiltin<"__builtin_ve_vl_vgtu_vvssml">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtu_vvssmvl : ClangBuiltin<"__builtin_ve_vl_vgtu_vvssmvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtunc_vvssl : ClangBuiltin<"__builtin_ve_vl_vgtunc_vvssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtunc_vvssvl : ClangBuiltin<"__builtin_ve_vl_vgtunc_vvssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtunc_vvssml : ClangBuiltin<"__builtin_ve_vl_vgtunc_vvssml">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtunc_vvssmvl : ClangBuiltin<"__builtin_ve_vl_vgtunc_vvssmvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlsx_vvssl : ClangBuiltin<"__builtin_ve_vl_vgtlsx_vvssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlsx_vvssvl : ClangBuiltin<"__builtin_ve_vl_vgtlsx_vvssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlsx_vvssml : ClangBuiltin<"__builtin_ve_vl_vgtlsx_vvssml">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlsx_vvssmvl : ClangBuiltin<"__builtin_ve_vl_vgtlsx_vvssmvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlsxnc_vvssl : ClangBuiltin<"__builtin_ve_vl_vgtlsxnc_vvssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlsxnc_vvssvl : ClangBuiltin<"__builtin_ve_vl_vgtlsxnc_vvssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
````
- **L1177 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1177 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1178 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1178 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1179 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1179 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1180 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1180 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1181 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1181 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1182 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1182 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1183 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1183 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1184 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1184 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1185 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1185 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1186 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1186 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1187 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1187 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1188 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1188 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1189 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1189 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1190 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1190 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1191 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1191 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1192 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1192 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1193 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1193 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1194 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1194 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1195 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1195 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1196 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1196 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1197 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1197 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1198 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1198 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1199 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1199 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1200 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1200 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1201-1224

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vgtlsxnc_vvssml : ClangBuiltin<"__builtin_ve_vl_vgtlsxnc_vvssml">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlsxnc_vvssmvl : ClangBuiltin<"__builtin_ve_vl_vgtlsxnc_vvssmvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlzx_vvssl : ClangBuiltin<"__builtin_ve_vl_vgtlzx_vvssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlzx_vvssvl : ClangBuiltin<"__builtin_ve_vl_vgtlzx_vvssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlzx_vvssml : ClangBuiltin<"__builtin_ve_vl_vgtlzx_vvssml">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlzx_vvssmvl : ClangBuiltin<"__builtin_ve_vl_vgtlzx_vvssmvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlzxnc_vvssl : ClangBuiltin<"__builtin_ve_vl_vgtlzxnc_vvssl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlzxnc_vvssvl : ClangBuiltin<"__builtin_ve_vl_vgtlzxnc_vvssvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlzxnc_vvssml : ClangBuiltin<"__builtin_ve_vl_vgtlzxnc_vvssml">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vgtlzxnc_vvssmvl : ClangBuiltin<"__builtin_ve_vl_vgtlzxnc_vvssmvl">, DefaultAttrsIntrinsic<[LLVMType<v256f64>], [LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<v256f64>, LLVMType<i32>], [IntrReadMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsc_vvssl : ClangBuiltin<"__builtin_ve_vl_vsc_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsc_vvssml : ClangBuiltin<"__builtin_ve_vl_vsc_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscnc_vvssl : ClangBuiltin<"__builtin_ve_vl_vscnc_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscnc_vvssml : ClangBuiltin<"__builtin_ve_vl_vscnc_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscot_vvssl : ClangBuiltin<"__builtin_ve_vl_vscot_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscot_vvssml : ClangBuiltin<"__builtin_ve_vl_vscot_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscncot_vvssl : ClangBuiltin<"__builtin_ve_vl_vscncot_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscncot_vvssml : ClangBuiltin<"__builtin_ve_vl_vscncot_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscu_vvssl : ClangBuiltin<"__builtin_ve_vl_vscu_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscu_vvssml : ClangBuiltin<"__builtin_ve_vl_vscu_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscunc_vvssl : ClangBuiltin<"__builtin_ve_vl_vscunc_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscunc_vvssml : ClangBuiltin<"__builtin_ve_vl_vscunc_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscuot_vvssl : ClangBuiltin<"__builtin_ve_vl_vscuot_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscuot_vvssml : ClangBuiltin<"__builtin_ve_vl_vscuot_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
````
- **L1201 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1201 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1202 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1202 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1203 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1203 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1204 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1204 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1205 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1205 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1206 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1206 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1207 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1207 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1208 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1208 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1209 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1209 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1210 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1210 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1211 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1211 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1212 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1212 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1213 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1213 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1214 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1214 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1215 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1215 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1216 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1216 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1217 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1217 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1218 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1218 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1219 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1219 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1220 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1220 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1221 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1221 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1222 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1222 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1223 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1223 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1224 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1224 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1225-1248

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_vscuncot_vvssl : ClangBuiltin<"__builtin_ve_vl_vscuncot_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscuncot_vvssml : ClangBuiltin<"__builtin_ve_vl_vscuncot_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscl_vvssl : ClangBuiltin<"__builtin_ve_vl_vscl_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vscl_vvssml : ClangBuiltin<"__builtin_ve_vl_vscl_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsclnc_vvssl : ClangBuiltin<"__builtin_ve_vl_vsclnc_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsclnc_vvssml : ClangBuiltin<"__builtin_ve_vl_vsclnc_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsclot_vvssl : ClangBuiltin<"__builtin_ve_vl_vsclot_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsclot_vvssml : ClangBuiltin<"__builtin_ve_vl_vsclot_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsclncot_vvssl : ClangBuiltin<"__builtin_ve_vl_vsclncot_vvssl">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_vsclncot_vvssml : ClangBuiltin<"__builtin_ve_vl_vsclncot_vvssml">, Intrinsic<[], [LLVMType<v256f64>, LLVMType<v256f64>, LLVMType<i64>, LLVMType<i64>, LLVMType<v256i1>, LLVMType<i32>], [IntrWriteMem]>;
let TargetPrefix = "ve" in def int_ve_vl_andm_mmm : ClangBuiltin<"__builtin_ve_vl_andm_mmm">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256i1>, LLVMType<v256i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_andm_MMM : ClangBuiltin<"__builtin_ve_vl_andm_MMM">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v512i1>, LLVMType<v512i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_orm_mmm : ClangBuiltin<"__builtin_ve_vl_orm_mmm">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256i1>, LLVMType<v256i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_orm_MMM : ClangBuiltin<"__builtin_ve_vl_orm_MMM">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v512i1>, LLVMType<v512i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_xorm_mmm : ClangBuiltin<"__builtin_ve_vl_xorm_mmm">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256i1>, LLVMType<v256i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_xorm_MMM : ClangBuiltin<"__builtin_ve_vl_xorm_MMM">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v512i1>, LLVMType<v512i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_eqvm_mmm : ClangBuiltin<"__builtin_ve_vl_eqvm_mmm">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256i1>, LLVMType<v256i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_eqvm_MMM : ClangBuiltin<"__builtin_ve_vl_eqvm_MMM">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v512i1>, LLVMType<v512i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_nndm_mmm : ClangBuiltin<"__builtin_ve_vl_nndm_mmm">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256i1>, LLVMType<v256i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_nndm_MMM : ClangBuiltin<"__builtin_ve_vl_nndm_MMM">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v512i1>, LLVMType<v512i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_negm_mm : ClangBuiltin<"__builtin_ve_vl_negm_mm">, Intrinsic<[LLVMType<v256i1>], [LLVMType<v256i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_negm_MM : ClangBuiltin<"__builtin_ve_vl_negm_MM">, Intrinsic<[LLVMType<v512i1>], [LLVMType<v512i1>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_pcvm_sml : ClangBuiltin<"__builtin_ve_vl_pcvm_sml">, Intrinsic<[LLVMType<i64>], [LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_lzvm_sml : ClangBuiltin<"__builtin_ve_vl_lzvm_sml">, Intrinsic<[LLVMType<i64>], [LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
````
- **L1225 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1225 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1226 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1226 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1227 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1227 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1228 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1228 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1229 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1229 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1230 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1230 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1231 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1231 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1232 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1232 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1233 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1233 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1234 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1234 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1235 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1235 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1236 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1236 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1237 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1237 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1238 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1238 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1239 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1239 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1240 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1240 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1241 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1241 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1242 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1242 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1243 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1243 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1244 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1244 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1245 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1245 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1246 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1246 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1247 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1247 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1248 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1248 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1249-1257

````tablegen
let TargetPrefix = "ve" in def int_ve_vl_tovm_sml : ClangBuiltin<"__builtin_ve_vl_tovm_sml">, Intrinsic<[LLVMType<i64>], [LLVMType<v256i1>, LLVMType<i32>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_lcr_sss : ClangBuiltin<"__builtin_ve_vl_lcr_sss">, Intrinsic<[LLVMType<i64>], [LLVMType<i64>, LLVMType<i64>], [IntrNoMem]>;
let TargetPrefix = "ve" in def int_ve_vl_scr_sss : ClangBuiltin<"__builtin_ve_vl_scr_sss">, Intrinsic<[], [LLVMType<i64>, LLVMType<i64>, LLVMType<i64>], [IntrNoMem, IntrHasSideEffects]>;
let TargetPrefix = "ve" in def int_ve_vl_tscr_ssss : ClangBuiltin<"__builtin_ve_vl_tscr_ssss">, Intrinsic<[LLVMType<i64>], [LLVMType<i64>, LLVMType<i64>, LLVMType<i64>], [IntrNoMem, IntrHasSideEffects]>;
let TargetPrefix = "ve" in def int_ve_vl_fidcr_sss : ClangBuiltin<"__builtin_ve_vl_fidcr_sss">, Intrinsic<[LLVMType<i64>], [LLVMType<i64>, LLVMType<i32>], [IntrNoMem, IntrHasSideEffects]>;
let TargetPrefix = "ve" in def int_ve_vl_fencei : ClangBuiltin<"__builtin_ve_vl_fencei">, Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;
let TargetPrefix = "ve" in def int_ve_vl_fencem_s : ClangBuiltin<"__builtin_ve_vl_fencem_s">, Intrinsic<[], [LLVMType<i32>], [IntrNoMem, IntrHasSideEffects]>;
let TargetPrefix = "ve" in def int_ve_vl_fencec_s : ClangBuiltin<"__builtin_ve_vl_fencec_s">, Intrinsic<[], [LLVMType<i32>], [IntrNoMem, IntrHasSideEffects]>;
let TargetPrefix = "ve" in def int_ve_vl_svob : ClangBuiltin<"__builtin_ve_vl_svob">, Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;
````
- **L1249 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1249 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1250 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1250 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1251 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1251 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1252 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1252 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1253 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1253 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1254 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1254 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1255 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1255 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1256 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1256 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1257 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1257 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- No direct include-style dependencies appear in this file. / 该文件中没有直接出现 include 风格依赖。

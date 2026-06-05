# Utility.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Conversion/TritonGPUToLLVM/Utility.h`
- **EN:** Provides helper utilities that support transformations or analysis in this area.
- **CN:** 提供支撑该领域分析或变换的辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_CONVERSION_TRITONGPU_TO_LLVM_UTILITY_H
   2: #define TRITON_CONVERSION_TRITONGPU_TO_LLVM_UTILITY_H
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-16
```cpp
   4: #include "mlir/Conversion/LLVMCommon/Pattern.h"
   5: #include "mlir/Dialect/LLVMIR/LLVMDialect.h"
   6: #include "mlir/Interfaces/FunctionInterfaces.h"
   7: #include "triton/Conversion/TritonGPUToLLVM/TargetInfoBase.h"
   8: #include "triton/Dialect/Triton/IR/Dialect.h"
   9: #include "triton/Dialect/Triton/IR/Utility.h"
  10: #include "triton/Dialect/TritonGPU/IR/Dialect.h"
  11: #include "triton/Dialect/TritonGPU/IR/LinearLayoutConversions.h"
  12: #include "triton/Dialect/TritonGPU/IR/Types.h"
  13: #include "triton/Tools/GenericSwizzling.h"
  14: #include "triton/Tools/LinearLayout.h"
  15: #include "triton/Tools/StrUtil.h"
  16: #include "llvm/ADT/STLExtras.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Conversion/LLVMCommon/Pattern.h, mlir/Dialect/LLVMIR/LLVMDialect.h, mlir/Interfaces/FunctionInterfaces.h, triton/Conversion/TritonGPUToLLVM/TargetInfoBase.h, triton/Dialect/Triton/IR/Dialect.h, and triton/Dialect/Triton/IR/Utility.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Conversion/LLVMCommon/Pattern.h, mlir/Dialect/LLVMIR/LLVMDialect.h, mlir/Interfaces/FunctionInterfaces.h, triton/Conversion/TritonGPUToLLVM/TargetInfoBase.h, triton/Dialect/Triton/IR/Dialect.h, and triton/Dialect/Triton/IR/Utility.h。

### Lines 18-18
```cpp
  18: #include <optional>
```
**EN:** This block imports the direct dependencies needed here, including <optional>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 <optional>。

### Lines 20-22
```cpp
  20: #define DEBUG_TYPE "ttgpu_to_llvm"
  21: #define DBGS() (llvm::dbgs() << "[" DEBUG_TYPE "]: ")
  22: #define LDBG(X) LLVM_DEBUG(DBGS() << X << "\n")
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 24-25
```cpp
  24: using namespace mlir;
  25: using namespace mlir::triton;
```
**EN:** This block stores supporting state such as mlir and triton, which other APIs in the file consume.
**CN:** 该代码块声明了 mlir and triton 等支撑状态，供本文件中的其他 API 使用。

### Lines 27-28
```cpp
  27: namespace mlir::LLVM {
  28: using namespace mlir::triton;
```
**EN:** This block stores supporting state such as triton, which other APIs in the file consume.
**CN:** 该代码块声明了 triton 等支撑状态，供本文件中的其他 API 使用。

### Lines 30-41
```cpp
  30: Value createConstantI1(Location loc, OpBuilder &rewriter, bool v);
  31: Value createConstantI32(Location loc, OpBuilder &rewriter, int32_t v);
  32: Value createConstantI64(Location loc, OpBuilder &rewriter, int64_t v);
  33: Value createConstantF16(Location loc, OpBuilder &rewriter, float v);
  34: Value createConstantBF16(Location loc, OpBuilder &rewriter, float v);
  35: Value createConstantF32(Location loc, OpBuilder &rewriter, float v);
  36: Value createConstantF64(Location loc, OpBuilder &rewriter, double v);
  37: Value createNaNConstant(Location loc, OpBuilder &rewriter, Type type);
  38: Value createIndexConstant(OpBuilder &builder, Location loc,
  39:                           const TypeConverter *converter, int64_t value);
  40: Value createLLVMIntegerConstant(OpBuilder &builder, Location loc, short width,
  41:                                 int64_t value);
```
**EN:** This block declares or defines callable APIs such as createConstantI1, createConstantI32, createConstantI64, createConstantF16, createConstantBF16, createConstantF32, createConstantF64, and createNaNConstant, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createConstantI1, createConstantI32, createConstantI64, createConstantF16, createConstantBF16, createConstantF32, createConstantF64, and createNaNConstant 等可调用 API，用来封装这里提供的核心行为。

### Lines 43-48
```cpp
  43: LLVM::CallOp createLLVMCallOp(OpBuilder &builder, Location loc,
  44:                               LLVMFuncOp funcOp, ValueRange args);
  45: LLVM::CallIntrinsicOp
  46: createLLVMIntrinsicCallOp(OpBuilder &builder, Location loc, StringRef intrinsic,
  47:                           TypeRange types, ValueRange args);
  48: } // namespace mlir::LLVM
```
**EN:** This block declares or defines callable APIs such as createLLVMCallOp and createLLVMIntrinsicCallOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createLLVMCallOp and createLLVMIntrinsicCallOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 50-50
```cpp
  50: namespace mlir::triton {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir::triton.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir::triton 下。

### Lines 52-54
```cpp
  52: struct TritonLLVMOpBuilder {
  53:   TritonLLVMOpBuilder(Location loc, OpBuilder &builder)
  54:       : loc(loc), builder(&builder) {}
```
**EN:** This block introduces `TritonLLVMOpBuilder`, the main class/struct defined here. Within the declaration, methods such as loc and builder expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `TritonLLVMOpBuilder`。 其中 loc and builder 等方法构成了它的主要接口。

### Lines 56-270
```cpp
  56:   // Shortcuts for some commonly used LLVM ops to keep code simple and intuitive
  57:   // Operators
  58:   template <typename... Args> LLVM::SIToFPOp inttofloat(Args &&...args) {
  59:     return LLVM::SIToFPOp::create(*builder, loc, std::forward<Args>(args)...);
  60:   }
  61:   template <typename... Args> LLVM::IntToPtrOp inttoptr(Args &&...args) {
  62:     return LLVM::IntToPtrOp::create(*builder, loc, std::forward<Args>(args)...);
  63:   }
  64:   template <typename... Args> LLVM::PtrToIntOp ptrtoint(Args &&...args) {
  65:     return LLVM::PtrToIntOp::create(*builder, loc, std::forward<Args>(args)...);
  66:   }
  67:   template <typename... Args> LLVM::ZExtOp zext(Args &&...args) {
  68:     return LLVM::ZExtOp::create(*builder, loc, std::forward<Args>(args)...);
  69:   }
  70:   template <typename... Args> LLVM::SExtOp sext(Args &&...args) {
  71:     return LLVM::SExtOp::create(*builder, loc, std::forward<Args>(args)...);
  72:   }
  73:   template <typename... Args> LLVM::FPExtOp fpext(Args &&...args) {
  74:     return LLVM::FPExtOp::create(*builder, loc, std::forward<Args>(args)...);
  75:   }
  76:   template <typename... Args> LLVM::FPTruncOp fptrunc(Args &&...args) {
  77:     return LLVM::FPTruncOp::create(*builder, loc, std::forward<Args>(args)...);
  78:   }
  79:   template <typename... Args> LLVM::TruncOp trunc(Args &&...args) {
  80:     return LLVM::TruncOp::create(*builder, loc, std::forward<Args>(args)...);
  81:   }
  82:   template <typename... Args> LLVM::UDivOp udiv(Args &&...args) {
  83:     return LLVM::UDivOp::create(*builder, loc, std::forward<Args>(args)...);
  84:   }
  85:   template <typename... Args> LLVM::SDivOp sdiv(Args &&...args) {
  86:     return LLVM::SDivOp::create(*builder, loc, std::forward<Args>(args)...);
  87:   }
  88:   template <typename... Args> LLVM::URemOp urem(Args &&...args) {
  89:     return LLVM::URemOp::create(*builder, loc, std::forward<Args>(args)...);
  90:   }
  91:   template <typename... Args> LLVM::AddOp add(Args &&...args) {
  92:     return LLVM::AddOp::create(*builder, loc, std::forward<Args>(args)...);
  93:   }
  94:   template <typename... Args> LLVM::SubOp sub(Args &&...args) {
  95:     return LLVM::SubOp::create(*builder, loc, std::forward<Args>(args)...);
  96:   }
  97:   template <typename... Args> LLVM::FAddOp fadd(Args &&...args) {
  98:     return LLVM::FAddOp::create(*builder, loc, std::forward<Args>(args)...);
  99:   }
 100:   template <typename... Args> LLVM::MulOp mul(Args &&...args) {
 101:     return LLVM::MulOp::create(*builder, loc, std::forward<Args>(args)...);
 102:   }
 103:   template <typename... Args> LLVM::FMulOp fmul(Args &&...args) {
 104:     return LLVM::FMulOp::create(*builder, loc, std::forward<Args>(args)...);
 105:   }
 106:   template <typename... Args> LLVM::FMAOp fma(Args &&...args) {
 107:     return LLVM::FMAOp::create(*builder, loc, std::forward<Args>(args)...);
 108:   }
 109:   template <typename... Args> LLVM::FNegOp neg(Args &&...args) {
 110:     return LLVM::FNegOp::create(*builder, loc, std::forward<Args>(args)...);
 111:   }
 112:   template <typename... Args> LLVM::SMaxOp smax(Args &&...args) {
 113:     return LLVM::SMaxOp::create(*builder, loc, std::forward<Args>(args)...);
 114:   }
 115:   template <typename... Args> LLVM::UMaxOp umax(Args &&...args) {
 116:     return LLVM::UMaxOp::create(*builder, loc, std::forward<Args>(args)...);
 117:   }
 118:   template <typename... Args> LLVM::MaxNumOp fmax(Args &&...args) {
 119:     return LLVM::MaxNumOp::create(*builder, loc, std::forward<Args>(args)...);
 120:   }
 121:   template <typename... Args> LLVM::SMinOp smin(Args &&...args) {
 122:     return LLVM::SMinOp::create(*builder, loc, std::forward<Args>(args)...);
 123:   }
 124:   template <typename... Args> LLVM::UMinOp umin(Args &&...args) {
 125:     return LLVM::UMinOp::create(*builder, loc, std::forward<Args>(args)...);
 126:   }
 127:   template <typename... Args> LLVM::MinNumOp fmin(Args &&...args) {
 128:     return LLVM::MinNumOp::create(*builder, loc, std::forward<Args>(args)...);
 129:   }
 130:   template <typename... Args> LLVM::ShlOp shl(Args &&...args) {
 131:     return LLVM::ShlOp::create(*builder, loc, std::forward<Args>(args)...);
 132:   }
 133:   template <typename... Args> LLVM::LShrOp lshr(Args &&...args) {
 134:     return LLVM::LShrOp::create(*builder, loc, std::forward<Args>(args)...);
 135:   }
 136:   template <typename... Args> LLVM::AShrOp ashr(Args &&...args) {
 137:     return LLVM::AShrOp::create(*builder, loc, std::forward<Args>(args)...);
 138:   }
 139:   template <typename... Args> LLVM::AndOp and_(Args &&...args) {
 140:     return LLVM::AndOp::create(*builder, loc, std::forward<Args>(args)...);
 141:   }
 142:   template <typename... Args> LLVM::XOrOp xor_(Args &&...args) {
 143:     return LLVM::XOrOp::create(*builder, loc, std::forward<Args>(args)...);
 144:   }
 145:   template <typename... Args> LLVM::OrOp or_(Args &&...args) {
 146:     return LLVM::OrOp::create(*builder, loc, std::forward<Args>(args)...);
 147:   }
 148:   LLVM::BitcastOp bitcast(Value val, Type type) {
 149:     return LLVM::BitcastOp::create(*builder, loc, type, val);
 150:   }
 151:   template <typename... Args>
 152:   LLVM::AddrSpaceCastOp addrspacecast(Args &&...args) {
 153:     return LLVM::AddrSpaceCastOp::create(*builder, loc,
 154:                                          std::forward<Args>(args)...);
 155:   }
 156:   template <typename... Args> LLVM::GEPOp gep(Args &&...args) {
 157:     return LLVM::GEPOp::create(*builder, loc, std::forward<Args>(args)...);
 158:   }
 159:   template <typename... Args> LLVM::InsertValueOp insert_val(Args &&...args) {
 160:     return LLVM::InsertValueOp::create(*builder, loc,
 161:                                        std::forward<Args>(args)...);
 162:   }
 163:   template <typename... Args> LLVM::ExtractValueOp extract_val(Args &&...args) {
 164:     return LLVM::ExtractValueOp::create(*builder, loc,
 165:                                         std::forward<Args>(args)...);
 166:   }
 167:   template <typename... Args>
 168:   LLVM::InsertElementOp insert_element(Args &&...args) {
 169:     return LLVM::InsertElementOp::create(*builder, loc,
 170:                                          std::forward<Args>(args)...);
 171:   }
 172:   template <typename... Args>
 173:   LLVM::ExtractElementOp extract_element(Args &&...args) {
 174:     return LLVM::ExtractElementOp::create(*builder, loc,
 175:                                           std::forward<Args>(args)...);
 176:   }
 177:   template <typename... Args> LLVM::LoadOp load(Args &&...args) {
 178:     return LLVM::LoadOp::create(*builder, loc, std::forward<Args>(args)...);
 179:   }
 180:   template <typename... Args> LLVM::StoreOp store(Args &&...args) {
 181:     return LLVM::StoreOp::create(*builder, loc, std::forward<Args>(args)...);
 182:   }
 183:   LLVM::FCmpOp fcmp_ogt(Value lhs, Value rhs) {
 184:     return LLVM::FCmpOp::create(*builder, loc, builder->getI1Type(),
 185:                                 LLVM::FCmpPredicate::ogt, lhs, rhs);
 186:   }
 187:   LLVM::FCmpOp fcmp_olt(Value lhs, Value rhs) {
 188:     return LLVM::FCmpOp::create(*builder, loc, builder->getI1Type(),
 189:                                 LLVM::FCmpPredicate::olt, lhs, rhs);
 190:   }
 191:   LLVM::FCmpOp fcmp_eq(Value lhs, Value rhs) {
 192:     return LLVM::FCmpOp::create(*builder, loc, builder->getI1Type(),
 193:                                 LLVM::FCmpPredicate::oeq, lhs, rhs);
 194:   }
 195:   template <typename... Args> LLVM::ICmpOp icmp_eq(Args &&...args) {
 196:     return LLVM::ICmpOp::create(*builder, loc, LLVM::ICmpPredicate::eq,
 197:                                 std::forward<Args>(args)...);
 198:   }
 199:   template <typename... Args> LLVM::ICmpOp icmp_ne(Args &&...args) {
 200:     return LLVM::ICmpOp::create(*builder, loc, LLVM::ICmpPredicate::ne,
 201:                                 std::forward<Args>(args)...);
 202:   }
 203:   template <typename... Args> LLVM::ICmpOp icmp_slt(Args &&...args) {
 204:     return LLVM::ICmpOp::create(*builder, loc, LLVM::ICmpPredicate::slt,
 205:                                 std::forward<Args>(args)...);
 206:   }
 207:   template <typename... Args> LLVM::ICmpOp icmp_sle(Args &&...args) {
 208:     return LLVM::ICmpOp::create(*builder, loc, LLVM::ICmpPredicate::sle,
 209:                                 std::forward<Args>(args)...);
 210:   }
 211:   template <typename... Args> LLVM::ICmpOp icmp_sgt(Args &&...args) {
 212:     return LLVM::ICmpOp::create(*builder, loc, LLVM::ICmpPredicate::sgt,
 213:                                 std::forward<Args>(args)...);
 214:   }
 215:   template <typename... Args> LLVM::ICmpOp icmp_sge(Args &&...args) {
 216:     return LLVM::ICmpOp::create(*builder, loc, LLVM::ICmpPredicate::sge,
 217:                                 std::forward<Args>(args)...);
 218:   }
 219:   template <typename... Args> LLVM::ICmpOp icmp_ult(Args &&...args) {
 220:     return LLVM::ICmpOp::create(*builder, loc, LLVM::ICmpPredicate::ult,
 221:                                 std::forward<Args>(args)...);
 222:   }
 223:   template <typename... Args> LLVM::ICmpOp icmp_ule(Args &&...args) {
 224:     return LLVM::ICmpOp::create(*builder, loc, LLVM::ICmpPredicate::ule,
 225:                                 std::forward<Args>(args)...);
 226:   }
 227:   template <typename... Args> LLVM::ICmpOp icmp_ugt(Args &&...args) {
 228:     return LLVM::ICmpOp::create(*builder, loc, LLVM::ICmpPredicate::ugt,
 229:                                 std::forward<Args>(args)...);
 230:   }
 231:   template <typename... Args> LLVM::ICmpOp icmp_uge(Args &&...args) {
 232:     return LLVM::ICmpOp::create(*builder, loc, LLVM::ICmpPredicate::uge,
 233:                                 std::forward<Args>(args)...);
 234:   }
 235:   template <typename... Args> LLVM::SelectOp select(Args &&...args) {
 236:     return LLVM::SelectOp::create(*builder, loc, std::forward<Args>(args)...);
 237:   }
 238:   template <typename... Args> LLVM::AddressOfOp address_of(Args &&...args) {
 239:     return LLVM::AddressOfOp::create(*builder, loc,
 240:                                      std::forward<Args>(args)...);
 241:   }
 242:   mlir::triton::gpu::BarrierOp barrier(triton::gpu::AddrSpace addrspace) {
 243:     return mlir::triton::gpu::BarrierOp::create(*builder, loc, addrspace);
 244:   }
 245:   template <typename... Args> LLVM::UndefOp undef(Args &&...args) {
 246:     return LLVM::UndefOp::create(*builder, loc, std::forward<Args>(args)...);
 247:   }
 248:   template <typename... Args> LLVM::ZeroOp null(Args &&...args) {
 249:     return LLVM::ZeroOp::create(*builder, loc, std::forward<Args>(args)...);
 250:   }
 251:   template <typename... Args> LLVM::CallOp call(Args &&...args) {
 252:     return LLVM::CallOp::create(*builder, loc, std::forward<Args>(args)...);
 253:   }
 254:   // Constants
 255:   Value int_val(short bitwidth, int64_t val) {
 256:     Type ty = builder->getIntegerType(bitwidth);
 257:     return LLVM::ConstantOp::create(*builder, loc, ty,
 258:                                     builder->getIntegerAttr(ty, val));
 259:   }
 260:   Value i1_val(int64_t val) { return int_val(1, val); }
 261:   Value true_val() { return int_val(1, true); }
 262:   Value false_val() { return int_val(1, false); }
 263:   Value f16_val(float v) { return LLVM::createConstantF16(loc, *builder, v); }
 264:   Value bf16_val(float v) { return LLVM::createConstantBF16(loc, *builder, v); }
 265:   Value f32_val(float v) { return LLVM::createConstantF32(loc, *builder, v); }
 266:   Value f64_val(double v) { return LLVM::createConstantF64(loc, *builder, v); }
 267:   Value i8_val(int64_t val) { return int_val(8, val); }
 268:   Value i16_val(int64_t val) { return int_val(16, val); }
 269:   Value i32_val(int64_t val) { return int_val(32, val); }
 270:   Value i64_val(int64_t val) { return int_val(64, val); }
```
**EN:** This block declares or defines callable APIs such as inttofloat, create, inttoptr, ptrtoint, zext, sext, fpext, and fptrunc, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inttofloat, create, inttoptr, ptrtoint, zext, sext, fpext, and fptrunc 等可调用 API，用来封装这里提供的核心行为。

### Lines 272-274
```cpp
 272:   Location loc;
 273:   OpBuilder *builder;
 274: };
```
**EN:** This block stores supporting state such as loc and builder, which other APIs in the file consume.
**CN:** 该代码块声明了 loc and builder 等支撑状态，供本文件中的其他 API 使用。

### Lines 276-286
```cpp
 276: // This builder combines an IRRewriter and a TritonLLVMOpBuilder into one,
 277: // making it easy to create operations with an implicit location and create LLVM
 278: // operations with shorthands.
 279: class TritonLLVMIRRewriter : public IRRewriter, public TritonLLVMOpBuilder {
 280: public:
 281:   // Create a builder with an implicit location. Arguments are forwarded to
 282:   // IRRewriter's constructor.
 283:   template <typename... Args>
 284:   TritonLLVMIRRewriter(Location loc, Args &&...args)
 285:       : IRRewriter(std::forward<Args>(args)...),
 286:         TritonLLVMOpBuilder(loc, *this) {}
```
**EN:** This block introduces `TritonLLVMIRRewriter`, the main class/struct defined here. Within the declaration, methods such as IRRewriter and TritonLLVMOpBuilder expose its core API. It also inherits behavior from a base type.
**CN:** 该代码块引入了此文件的核心类/结构体 `TritonLLVMIRRewriter`。 其中 IRRewriter and TritonLLVMOpBuilder 等方法构成了它的主要接口。 它还通过继承复用基类能力。

### Lines 288-291
```cpp
 288:   // Get the implicit location.
 289:   Location getLoc() const { return loc; }
 290:   // Set the implicit location used to build ops.
 291:   void setLoc(Location loc) { this->loc = loc; }
```
**EN:** This block declares or defines callable APIs such as getLoc and setLoc, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getLoc and setLoc 等可调用 API，用来封装这里提供的核心行为。

### Lines 293-298
```cpp
 293:   // Wrapper for op creation that passes an implicit location.
 294:   template <typename OpTy, typename... Args> OpTy create(Args &&...args) {
 295:     return OpBuilder::create<OpTy>(loc, std::forward<Args>(args)...);
 296:   }
 297: };
 298: } // namespace mlir::triton
```
**EN:** This block declares or defines callable APIs such as create, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 create 等可调用 API，用来封装这里提供的核心行为。

### Lines 300-317
```cpp
 300: // Types
 301: #define ptr_ty(...) LLVM::LLVMPointerType::get(__VA_ARGS__)
 302: #define int_ty(width) rewriter.getIntegerType(width)
 303: #define i16_ty rewriter.getIntegerType(16)
 304: #define i32_ty rewriter.getIntegerType(32)
 305: #define i64_ty rewriter.getIntegerType(64)
 306: #define ui32_ty rewriter.getIntegerType(32, false)
 307: #define ui64_ty rewriter.getIntegerType(64, false)
 308: #define f16_ty rewriter.getF16Type()
 309: #define bf16_ty rewriter.getBF16Type()
 310: #define i8_ty rewriter.getIntegerType(8)
 311: #define i1_ty rewriter.getI1Type()
 312: #define f32_ty rewriter.getF32Type()
 313: #define f64_ty rewriter.getF64Type()
 314: #define vec_ty(type, num) VectorType::get(num, type)
 315: #define void_ty(ctx) LLVM::LLVMVoidType::get(ctx)
 316: #define struct_ty(...) LLVM::LLVMStructType::getLiteral(ctx, __VA_ARGS__)
 317: #define array_ty(elemTy, count) LLVM::LLVMArrayType::get(elemTy, count)
```
**EN:** This block declares or defines callable APIs such as ptr_ty, get, int_ty, getIntegerType, getF16Type, getBF16Type, getI1Type, and getF32Type, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ptr_ty, get, int_ty, getIntegerType, getF16Type, getBF16Type, getI1Type, and getF32Type 等可调用 API，用来封装这里提供的核心行为。

### Lines 319-322
```cpp
 319: // Attributes
 320: #define i32_arr_attr(...) rewriter.getI32ArrayAttr({__VA_ARGS__})
 321: #define i64_arr_attr(...) rewriter.getI64ArrayAttr({__VA_ARGS__})
 322: #define str_attr(str) ::mlir::StringAttr::get(ctx, (str))
```
**EN:** This block declares or defines callable APIs such as i32_arr_attr, getI32ArrayAttr, i64_arr_attr, getI64ArrayAttr, and str_attr, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 i32_arr_attr, getI32ArrayAttr, i64_arr_attr, getI64ArrayAttr, and str_attr 等可调用 API，用来封装这里提供的核心行为。

### Lines 324-324
```cpp
 324: namespace mlir {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir 下。

### Lines 326-329
```cpp
 326: // See FuncOpToLLVM.cpp for details about Triton's function calling conventions
 327: constexpr int kProfileScratchBufferOffset = -1;
 328: constexpr int kGlobalScratchBufferOffset = -2;
 329: constexpr int kSharedMemoryOffset = -3;
```
**EN:** This block defines named compile-time constants such as See, FuncOpToLLVM, cpp, for, details, and about.
**CN:** 该代码块定义了 See, FuncOpToLLVM, cpp, for, details, and about 等具名编译期常量。

### Lines 331-331
```cpp
 331: namespace triton {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under triton.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 triton 下。

### Lines 333-333
```cpp
 333: namespace gpu {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under gpu.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 gpu 下。

### Lines 335-336
```cpp
 335: std::pair<SmallVector<LocalMemOpTile>, SmallVector<LocalMemOpTile>>
 336: getSrcDstTiles(const TargetInfoBase &targetInfo, int bitwidth, bool crossCTA);
```
**EN:** This block declares or defines callable APIs such as getSrcDstTiles, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSrcDstTiles 等可调用 API，用来封装这里提供的核心行为。

### Lines 338-338
```cpp
 338: Type getFunctionType(Type resultType, ValueRange operands);
```
**EN:** This block declares or defines callable APIs such as getFunctionType, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getFunctionType 等可调用 API，用来封装这里提供的核心行为。

### Lines 340-343
```cpp
 340: LLVM::LLVMFuncOp appendOrGetExternFuncOp(RewriterBase &rewriter, Operation *op,
 341:                                          StringRef funcName, Type funcType,
 342:                                          StringRef libname = "",
 343:                                          StringRef libpath = "");
```
**EN:** This block declares or defines callable APIs such as appendOrGetExternFuncOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 appendOrGetExternFuncOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 345-346
```cpp
 345: // Multiply a square layout with 1 input and output dimension with a vector
 346: Value matrixVectorProd(TritonLLVMOpBuilder &b, const LinearLayout &A, Value x);
```
**EN:** This block declares or defines callable APIs such as matrixVectorProd, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 matrixVectorProd 等可调用 API，用来封装这里提供的核心行为。

### Lines 348-349
```cpp
 348: // Whether the convert layout should be forced to use warp shuffles.
 349: bool cvtAlwaysUseWarpShuffle(triton::gpu::ConvertLayoutOp cvt);
```
**EN:** This block declares or defines callable APIs such as cvtAlwaysUseWarpShuffle, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 cvtAlwaysUseWarpShuffle 等可调用 API，用来封装这里提供的核心行为。

### Lines 351-357
```cpp
 351: // Return a predicate that is true only if the current thread holds unique data,
 352: // according to freeVarsMask. The predicate may be null to indicate no
 353: // predication is required.
 354: Value emitRedundantThreadPredicate(
 355:     const llvm::MapVector<StringAttr, int32_t> &freeVarMasks,
 356:     ConversionPatternRewriter &rewriter, Location loc,
 357:     const TargetInfoBase &targetInfo);
```
**EN:** This block declares or defines callable APIs such as emitRedundantThreadPredicate, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 emitRedundantThreadPredicate 等可调用 API，用来封装这里提供的核心行为。

### Lines 359-360
```cpp
 359: // Takes two values that may be boolean, or null to represent constant True.
 360: Value maybeAnd(OpBuilder &builder, Location loc, Value a, Value b);
```
**EN:** This block declares or defines callable APIs such as maybeAnd, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 maybeAnd 等可调用 API，用来封装这里提供的核心行为。

### Lines 362-362
```cpp
 362: } // namespace gpu
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 364-364
```cpp
 364: } // namespace triton
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 366-367
```cpp
 366: namespace LLVM {
 367: using namespace mlir::triton;
```
**EN:** This block stores supporting state such as triton, which other APIs in the file consume.
**CN:** 该代码块声明了 triton 等支撑状态，供本文件中的其他 API 使用。

### Lines 369-377
```cpp
 369: /// Represents a shared memory allocation for tensor operations.
 370: ///
 371: /// For non-partitioned tensors, this contains a single base pointer.
 372: /// For partitioned tensors (PartitionedEncodingAttr), this contains multiple
 373: /// base pointers, one per partition.
 374: class SharedMemoryObject {
 375: public:
 376:   /// Single-base constructor (for non-partitioned tensors)
 377:   SharedMemoryObject(Value base, Type baseElemType, ArrayRef<Value> offsets);
```
**EN:** This block introduces `SharedMemoryObject`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `SharedMemoryObject`。

### Lines 379-381
```cpp
 379:   /// Multi-base constructor (for partitioned tensors)
 380:   SharedMemoryObject(ArrayRef<Value> bases, Type baseElemType,
 381:                      ArrayRef<Value> offsets);
```
**EN:** This block declares or defines callable APIs such as SharedMemoryObject, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 SharedMemoryObject 等可调用 API，用来封装这里提供的核心行为。

### Lines 383-385
```cpp
 383:   /// Single-base constructor
 384:   SharedMemoryObject(Value base, Type baseElemType, int64_t rank, Location loc,
 385:                      RewriterBase &rewriter);
```
**EN:** This block declares or defines callable APIs such as SharedMemoryObject, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 SharedMemoryObject 等可调用 API，用来封装这里提供的核心行为。

### Lines 387-389
```cpp
 387:   /// Multi-base constructor with zero offsets
 388:   SharedMemoryObject(ArrayRef<Value> bases, Type baseElemType, int64_t rank,
 389:                      Location loc, RewriterBase &rewriter);
```
**EN:** This block declares or defines callable APIs such as SharedMemoryObject, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 SharedMemoryObject 等可调用 API，用来封装这里提供的核心行为。

### Lines 391-391
```cpp
 391:   SmallVector<Value> getOffsets() const { return offsets; }
```
**EN:** This block declares or defines callable APIs such as getOffsets, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOffsets 等可调用 API，用来封装这里提供的核心行为。

### Lines 393-402
```cpp
 393:   /// Returns the single base pointer.
 394:   /// IMPORTANT: This asserts that there is exactly one base. For partitioned
 395:   /// tensors (multiple bases), use getBases() instead.
 396:   /// Callers should use getBases() and handle all partitions appropriately.
 397:   Value getBase() const {
 398:     assert(bases.size() == 1 &&
 399:            "getBase() called on partitioned tensor with multiple bases. "
 400:            "Use getBases() and handle all partitions.");
 401:     return bases[0];
 402:   }
```
**EN:** This block declares or defines callable APIs such as getBase, size, and getBases, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBase, size, and getBases 等可调用 API，用来封装这里提供的核心行为。

### Lines 404-406
```cpp
 404:   /// Returns all base pointers. For partitioned tensors, returns one base
 405:   /// per partition.
 406:   ArrayRef<Value> getBases() const { return bases; }
```
**EN:** This block declares or defines callable APIs such as getBases, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBases 等可调用 API，用来封装这里提供的核心行为。

### Lines 408-410
```cpp
 408:   /// Returns the number of base pointers (1 for non-partitioned, N for
 409:   /// partitioned).
 410:   size_t getNumBases() const { return bases.size(); }
```
**EN:** This block declares or defines callable APIs such as getNumBases and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNumBases and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 412-412
```cpp
 412:   Type getBaseElemType() const { return baseElemType; }
```
**EN:** This block declares or defines callable APIs such as getBaseElemType, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBaseElemType 等可调用 API，用来封装这里提供的核心行为。

### Lines 414-414
```cpp
 414:   SmallVector<Value> getElems() const;
```
**EN:** This block declares or defines callable APIs such as getElems, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getElems 等可调用 API，用来封装这里提供的核心行为。

### Lines 416-416
```cpp
 416:   SmallVector<Type> getTypes() const;
```
**EN:** This block declares or defines callable APIs such as getTypes, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getTypes 等可调用 API，用来封装这里提供的核心行为。

### Lines 418-422
```cpp
 418:   // Returns a mask representing all the bits of the memdesc offsets that
 419:   // may be modified by an affine offset coming from a memdesc_subslice.
 420:   // The offsets are considered to be in the type of the memdesc.
 421:   // For padded layouts, we return the offsets without padding.
 422:   static uint64_t getMaskSpanOffsets(triton::gpu::MemDescType srcTy);
```
**EN:** This block declares or defines callable APIs such as getMaskSpanOffsets, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getMaskSpanOffsets 等可调用 API，用来封装这里提供的核心行为。

### Lines 424-428
```cpp
 424:   // Returns whether the shared memory access had a memdesc_subslice
 425:   // that is rank-preserving (soon to be called memdesc_slice)
 426:   static bool isAffineSharedMemoryAccess(triton::gpu::MemDescType srcTy) {
 427:     return getMaskSpanOffsets(srcTy) != 0;
 428:   }
```
**EN:** This block declares or defines callable APIs such as isAffineSharedMemoryAccess and getMaskSpanOffsets, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isAffineSharedMemoryAccess and getMaskSpanOffsets 等可调用 API，用来封装这里提供的核心行为。

### Lines 430-433
```cpp
 430:   Value getShmemOffset(Location loc, RewriterBase &rewriter,
 431:                        triton::gpu::MemDescType srcTy) const;
 432:   Value getShmemAffineBase(Location loc, RewriterBase &rewriter,
 433:                            triton::gpu::MemDescType srcTy) const;
```
**EN:** This block declares or defines callable APIs such as getShmemOffset and getShmemAffineBase, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getShmemOffset and getShmemAffineBase 等可调用 API，用来封装这里提供的核心行为。

### Lines 435-439
```cpp
 435:   // TODO(Keren): deprecate the method once AMD backend has cleaned up
 436:   Value getCSwizzleOffset(int dim) const {
 437:     assert(dim >= 0 && dim < offsets.size());
 438:     return offsets[dim];
 439:   }
```
**EN:** This block declares or defines callable APIs such as getCSwizzleOffset and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getCSwizzleOffset and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 441-448
```cpp
 441: private:
 442:   SmallVector<Value>
 443:       bases; // Shared memory base pointers. One for non-partitioned tensors,
 444:              // multiple for partitioned tensors (one per partition).
 445:   Type baseElemType;
 446:   SmallVector<Value>
 447:       offsets; // i32 int. The offsets are zero at the initial allocation.
 448: };
```
**EN:** This block stores supporting state such as bases, baseElemType, and offsets, which other APIs in the file consume.
**CN:** 该代码块声明了 bases, baseElemType, and offsets 等支撑状态，供本文件中的其他 API 使用。

### Lines 450-452
```cpp
 450: Value getStructFromSharedMemoryObject(Location loc,
 451:                                       const SharedMemoryObject &smemObj,
 452:                                       RewriterBase &rewriter);
```
**EN:** This block declares or defines callable APIs such as getStructFromSharedMemoryObject, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getStructFromSharedMemoryObject 等可调用 API，用来封装这里提供的核心行为。

### Lines 454-457
```cpp
 454: SharedMemoryObject getSharedMemoryObjectFromStruct(Location loc,
 455:                                                    Value llvmStruct,
 456:                                                    Type elemTy,
 457:                                                    RewriterBase &rewriter);
```
**EN:** This block declares or defines callable APIs such as getSharedMemoryObjectFromStruct, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSharedMemoryObjectFromStruct 等可调用 API，用来封装这里提供的核心行为。

### Lines 459-462
```cpp
 459: // Build a vector of shared-memory base pointers for dynamic partition
 460: // indexing (expects at least two bases).
 461: Value buildBasePtrVector(Location loc, RewriterBase &rewriter,
 462:                          ArrayRef<Value> smemBases);
```
**EN:** This block declares or defines callable APIs such as buildBasePtrVector, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 buildBasePtrVector 等可调用 API，用来封装这里提供的核心行为。

### Lines 464-468
```cpp
 464: // Convert an \param index to a multi-dim coordinate given \param shape and
 465: // \param order.
 466: SmallVector<Value> delinearize(RewriterBase &rewriter, Location loc,
 467:                                Value linear, ArrayRef<unsigned> shape,
 468:                                ArrayRef<unsigned> order);
```
**EN:** This block declares or defines callable APIs such as delinearize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 delinearize 等可调用 API，用来封装这里提供的核心行为。

### Lines 470-471
```cpp
 470: SmallVector<Value> delinearize(RewriterBase &rewriter, Location loc,
 471:                                unsigned linear, ArrayRef<unsigned> shape);
```
**EN:** This block declares or defines callable APIs such as delinearize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 delinearize 等可调用 API，用来封装这里提供的核心行为。

### Lines 473-474
```cpp
 473: SmallVector<Value> delinearize(RewriterBase &rewriter, Location loc,
 474:                                Value linear, ArrayRef<unsigned> shape);
```
**EN:** This block declares or defines callable APIs such as delinearize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 delinearize 等可调用 API，用来封装这里提供的核心行为。

### Lines 476-477
```cpp
 476: SmallVector<unsigned> delinearize(unsigned linear, ArrayRef<unsigned> shape,
 477:                                   ArrayRef<unsigned> order);
```
**EN:** This block declares or defines callable APIs such as delinearize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 delinearize 等可调用 API，用来封装这里提供的核心行为。

### Lines 479-486
```cpp
 479: // Returns a tuple with the delinearized coordinates and a boolean which is true
 480: // iff the Value is not broadcasted (equivalently, if the value is the "first"
 481: // lane/thread/etc. that holds the given value). In mathy terms, the boolean is
 482: // true if the element is the canonical representative of the class.
 483: std::tuple<SmallVector<Value>, Value>
 484: delinearize(RewriterBase &rewriter, Location loc,
 485:             triton::gpu::DistributedEncodingTrait layout,
 486:             ArrayRef<int64_t> shape, StringAttr dimName, Value linear);
```
**EN:** This block declares or defines callable APIs such as delinearize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 delinearize 等可调用 API，用来封装这里提供的核心行为。

### Lines 488-489
```cpp
 488: Value linearize(RewriterBase &rewriter, Location loc, ArrayRef<Value> multiDim,
 489:                 ArrayRef<unsigned> shape, ArrayRef<unsigned> order);
```
**EN:** This block declares or defines callable APIs such as linearize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 linearize 等可调用 API，用来封装这里提供的核心行为。

### Lines 491-492
```cpp
 491: Value linearize(RewriterBase &rewriter, Location loc, ArrayRef<Value> multiDim,
 492:                 ArrayRef<unsigned> shape);
```
**EN:** This block declares or defines callable APIs such as linearize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 linearize 等可调用 API，用来封装这里提供的核心行为。

### Lines 494-495
```cpp
 494: Value linearize(RewriterBase &rewriter, Location loc, ArrayRef<Value> multiDim,
 495:                 triton::gpu::LinearEncodingAttr encoding, StringAttr dimName);
```
**EN:** This block declares or defines callable APIs such as linearize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 linearize 等可调用 API，用来封装这里提供的核心行为。

### Lines 497-498
```cpp
 497: size_t linearize(ArrayRef<unsigned> multiDim, ArrayRef<unsigned> shape,
 498:                  ArrayRef<unsigned> order);
```
**EN:** This block declares or defines callable APIs such as linearize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 linearize 等可调用 API，用来封装这里提供的核心行为。

### Lines 500-501
```cpp
 500: GlobalOp getOrInsertGlobalConstant(RewriterBase &rewriter, ModuleOp module,
 501:                                    Type type, Attribute content, StringRef key);
```
**EN:** This block declares or defines callable APIs such as getOrInsertGlobalConstant, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOrInsertGlobalConstant 等可调用 API，用来封装这里提供的核心行为。

### Lines 503-504
```cpp
 503: Value addStringToModule(Location loc, RewriterBase &rewriter, StringRef key,
 504:                         StringRef content);
```
**EN:** This block declares or defines callable APIs such as addStringToModule, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 addStringToModule 等可调用 API，用来封装这里提供的核心行为。

### Lines 506-506
```cpp
 506: Value getStackPointer(RewriterBase &rewriter, FunctionOpInterface funcOp);
```
**EN:** This block declares or defines callable APIs such as getStackPointer, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getStackPointer 等可调用 API，用来封装这里提供的核心行为。

### Lines 508-510
```cpp
 508: Value getGlobalScratchPtr(Location loc, RewriterBase &rewriter,
 509:                           const TargetInfoBase &targetInfo,
 510:                           FunctionOpInterface funcOp, Value allocOffset = {});
```
**EN:** This block declares or defines callable APIs such as getGlobalScratchPtr, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getGlobalScratchPtr 等可调用 API，用来封装这里提供的核心行为。

### Lines 512-515
```cpp
 512: Value getProfileScratchPtr(Location loc, RewriterBase &rewriter,
 513:                            const TargetInfoBase &targetInfo,
 514:                            FunctionOpInterface funcOp, Value allocOffset = {},
 515:                            bool currentCTA = true);
```
**EN:** This block declares or defines callable APIs such as getProfileScratchPtr, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getProfileScratchPtr 等可调用 API，用来封装这里提供的核心行为。

### Lines 517-518
```cpp
 517: Value getSharedMemoryBase(Location loc, RewriterBase &rewriter,
 518:                           const TargetInfoBase &target, Operation *op);
```
**EN:** This block declares or defines callable APIs such as getSharedMemoryBase, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSharedMemoryBase 等可调用 API，用来封装这里提供的核心行为。

### Lines 520-523
```cpp
 520: /// Returns the allocation offsets for the given operation.
 521: /// For non-partitioned tensors, returns a single offset.
 522: /// For partitioned tensors, returns multiple offsets (one per partition).
 523: SmallVector<int64_t> getPartitionOffsets(Operation *op);
```
**EN:** This block declares or defines callable APIs such as getPartitionOffsets, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getPartitionOffsets 等可调用 API，用来封装这里提供的核心行为。

### Lines 525-531
```cpp
 525: /// Returns all shared memory bases for the given operation.
 526: /// For non-partitioned tensors, returns a single base pointer.
 527: /// For partitioned tensors, returns multiple base pointers (one per
 528: /// partition).
 529: SmallVector<Value> getSharedMemoryBases(Location loc, RewriterBase &rewriter,
 530:                                         const TargetInfoBase &target,
 531:                                         Operation *op);
```
**EN:** This block declares or defines callable APIs such as getSharedMemoryBases, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getSharedMemoryBases 等可调用 API，用来封装这里提供的核心行为。

### Lines 533-535
```cpp
 533: // -----------------------------------------------------------------------
 534: // MXFP utilities
 535: // -----------------------------------------------------------------------
```
**EN:** This comment block records the intent and constraints of the surrounding code: ----------------------------------------------------------------------- MXFP utilities -----------------------------------------------------------------------.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 537-539
```cpp
 537: // Scale a mxfp4 value by a given scale.
 538: Value mxfpScaleBf16(RewriterBase &rewriter, Location loc, Value v, Value scale,
 539:                     bool fastMath);
```
**EN:** This block declares or defines callable APIs such as mxfpScaleBf16, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 mxfpScaleBf16 等可调用 API，用来封装这里提供的核心行为。

### Lines 541-541
```cpp
 541: } // namespace LLVM
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 543-545
```cpp
 543: // -----------------------------------------------------------------------
 544: // Hardware Indices
 545: // -----------------------------------------------------------------------
```
**EN:** This comment block records the intent and constraints of the surrounding code: ----------------------------------------------------------------------- Hardware Indices -----------------------------------------------------------------------.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 547-549
```cpp
 547: // If an operation is contained within a warp specialize region, this returns
 548: // the warp ID offset of that warpgroup.
 549: std::optional<int> getWarpGroupStartWarpId(Block *block);
```
**EN:** This block declares or defines callable APIs such as getWarpGroupStartWarpId, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getWarpGroupStartWarpId 等可调用 API，用来封装这里提供的核心行为。

### Lines 551-553
```cpp
 551: // If an operation is contained within a warp specialize region, this returns
 552: // the thread ID offset of that warpgroup.
 553: std::optional<int> getWarpGroupStartThreadId(Block *block);
```
**EN:** This block declares or defines callable APIs such as getWarpGroupStartThreadId, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getWarpGroupStartThreadId 等可调用 API，用来封装这里提供的核心行为。

### Lines 555-556
```cpp
 555: // Returns CTA level thread ID.
 556: Value getThreadId(OpBuilder &rewriter, Location loc);
```
**EN:** This block declares or defines callable APIs such as getThreadId, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getThreadId 等可调用 API，用来封装这里提供的核心行为。

### Lines 558-559
```cpp
 558: // Get the lane ID, which is index of the thread within its warp.
 559: Value getLaneId(OpBuilder &rewriter, Location loc);
```
**EN:** This block declares or defines callable APIs such as getLaneId, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getLaneId 等可调用 API，用来封装这里提供的核心行为。

### Lines 561-562
```cpp
 561: // Get the lane ID and warp ID.
 562: std::pair<Value, Value> getLaneAndWarpId(OpBuilder &rewriter, Location loc);
```
**EN:** This block declares or defines callable APIs such as getLaneAndWarpId, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getLaneAndWarpId 等可调用 API，用来封装这里提供的核心行为。

### Lines 564-574
```cpp
 564: // -----------------------------------------------------------------------
 565: // Shared memory utilities
 566: // -----------------------------------------------------------------------
 567: using LLVM::SharedMemoryObject;
 568: using ::mlir::LLVM::delinearize;
 569: using ::mlir::triton::gpu::AMDMfmaEncodingAttr;
 570: using ::mlir::triton::gpu::AMDWmmaEncodingAttr;
 571: using ::mlir::triton::gpu::BlockedEncodingAttr;
 572: using ::mlir::triton::gpu::DotOperandEncodingAttr;
 573: using ::mlir::triton::gpu::NvidiaMmaEncodingAttr;
 574: using ::mlir::triton::gpu::SliceEncodingAttr;
```
**EN:** This block stores supporting state such as SharedMemoryObject, delinearize, AMDMfmaEncodingAttr, AMDWmmaEncodingAttr, BlockedEncodingAttr, and DotOperandEncodingAttr, which other APIs in the file consume.
**CN:** 该代码块声明了 SharedMemoryObject, delinearize, AMDMfmaEncodingAttr, AMDWmmaEncodingAttr, BlockedEncodingAttr, and DotOperandEncodingAttr 等支撑状态，供本文件中的其他 API 使用。

### Lines 576-577
```cpp
 576: Value dot(RewriterBase &rewriter, Location loc, ArrayRef<Value> offsets,
 577:           ArrayRef<Value> strides);
```
**EN:** This block declares or defines callable APIs such as dot, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 dot 等可调用 API，用来封装这里提供的核心行为。

### Lines 579-587
```cpp
 579: // "Applies" the given layout by computing layout(indices) and returning the
 580: // resulting Values.
 581: //
 582: // In other words, this generates LLVM-dialect MLIR code to "run" the layout
 583: // function.
 584: SmallVector<std::pair<StringAttr, Value>>
 585: applyLinearLayout(Location loc, RewriterBase &rewriter,
 586:                   const LinearLayout &layout,
 587:                   ArrayRef<std::pair<StringAttr, Value>> indices);
```
**EN:** This block declares or defines callable APIs such as applyLinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 applyLinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 589-590
```cpp
 589: SmallVector<SmallVector<unsigned>> emitOffsetForLayout(Attribute layout,
 590:                                                        RankedTensorType type);
```
**EN:** This block declares or defines callable APIs such as emitOffsetForLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 emitOffsetForLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 592-601
```cpp
 592: // Emit indices calculation within each ConversionPattern, and returns a
 593: // [elemsPerThread X rank] index matrix.
 594: //
 595: // For example, for a thread a owns `elemsPerThread` elements of a tensor with
 596: // type `type` and layout `layout`, the result will contain `elemsPerThread`
 597: // vectors. Each vector contains the SSA values of the indices required to
 598: // access the corresponding element, starting from the inner dimension.
 599: SmallVector<SmallVector<Value>>
 600: emitIndices(Location loc, RewriterBase &rewriter, const TargetInfoBase &target,
 601:             Attribute layout, RankedTensorType type, bool withCTAOffset);
```
**EN:** This block declares or defines callable APIs such as emitIndices, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 emitIndices 等可调用 API，用来封装这里提供的核心行为。

### Lines 603-606
```cpp
 603: SmallVector<SmallVector<Value>>
 604: emitIndices(Location loc, RewriterBase &rewriter, const TargetInfoBase &target,
 605:             const LinearLayout &layout, RankedTensorType type,
 606:             bool withCTAOffset);
```
**EN:** This block declares or defines callable APIs such as emitIndices, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 emitIndices 等可调用 API，用来封装这里提供的核心行为。

### Lines 608-616
```cpp
 608: // Compute per-element shared-memory pointers for a local atomic/ldst update by
 609: // replacing `coords[*][axis]` with `idxValues[*]` and mapping the resulting
 610: // logical coordinates back to shared-memory offsets.
 611: SmallVector<Value> computeLocalPtrs(Location loc,
 612:                                     triton::gpu::MemDescType memDescTy,
 613:                                     SharedMemoryObject smemObj, Type llvmElemTy,
 614:                                     ArrayRef<Value> idxValues,
 615:                                     ArrayRef<SmallVector<Value>> coords,
 616:                                     unsigned axis, RewriterBase &rewriter);
```
**EN:** This block declares or defines callable APIs such as computeLocalPtrs, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 computeLocalPtrs 等可调用 API，用来封装这里提供的核心行为。

### Lines 618-622
```cpp
 618: // Calculates the required interval chunking and padding logical-shift values
 619: // for shared memory padding, depending on elements' bit width and whether
 620: // offsets count the number of bytes or number of elements.
 621: SmallVector<std::pair<unsigned, unsigned>>
 622: getPaddedSharedShifts(Attribute enc, unsigned bitwidth, bool offsetInBytes);
```
**EN:** This block declares or defines callable APIs such as getPaddedSharedShifts, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getPaddedSharedShifts 等可调用 API，用来封装这里提供的核心行为。

### Lines 624-628
```cpp
 624: // Applies padding to base offset values in shared memory.
 625: Value applyPadding(Location loc, RewriterBase &rewriter, Value baseOffset,
 626:                    ArrayRef<std::pair<unsigned, unsigned>> shifts);
 627: uint32_t applyPadding(uint32_t baseOffset,
 628:                       ArrayRef<std::pair<unsigned, unsigned>> shifts);
```
**EN:** This block declares or defines callable APIs such as applyPadding, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 applyPadding 等可调用 API，用来封装这里提供的核心行为。

### Lines 630-644
```cpp
 630: // Close cousin of lowerLdStMatrix in MemoryOpToLLVM.cpp
 631: // We might want to merge them at some point, but having to support
 632: // ldmatrix.trans makes the code in lowerLdStMatrix a bit specific
 633: // Lowers to st when valArrays is empty, and to ld when it is not,
 634: // and returns the output values.
 635: // `paddingShifts` encodes shared memory padding if any.
 636: SmallVector<Value>
 637: lowerLdStShared(Location loc, MLIRContext *ctx, LinearLayout cvt,
 638:                 ArrayRef<Value> valsArray, // Input for store, output for load
 639:                 Type llvmElemTy, ArrayRef<Value> smemBases,
 640:                 ArrayRef<std::pair<unsigned, unsigned>> paddingShifts,
 641:                 Value affineOffset, uint64_t maskSpanAffineOffset,
 642:                 RewriterBase &rewriter, const TargetInfoBase &targetInfo,
 643:                 std::optional<int> maybeMaxVecElems = {},
 644:                 Operation *localLoadOp = nullptr);
```
**EN:** This block declares or defines callable APIs such as lowerLdStShared, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 lowerLdStShared 等可调用 API，用来封装这里提供的核心行为。

### Lines 646-664
```cpp
 646: // Lower an ld/st-like operation given a layout and a callback that creates the
 647: // PTX instruction Lowers to st when valArrays is empty, and to ld when it is
 648: // not, and returns the output values.
 649: // calcPaddedOffset is a lambda that takes a base offset (mlir::Value)
 650: // and computes a new offset (mlir::Value) by applying padding based on
 651: // shared memory layout.
 652: // cvt: Maps (reg, lane, warp, block) → (offset[, partition]).
 653: SmallVector<Value>
 654: lowerLdSt(Location loc, MLIRContext *ctx, LinearLayout cvt,
 655:           ArrayRef<Value> valsArray, // Input for store, output for load
 656:           Type llvmElemTy, ArrayRef<Value> smemBases,
 657:           ArrayRef<std::pair<unsigned, unsigned>> paddingShifts,
 658:           Value affineOffset, uint64_t maskSpanAffineOffset, Value laneId,
 659:           Value warpId, RewriterBase &rewriter,
 660:           const TargetInfoBase &targetInfo, std::optional<int> maybeMaxVecElems,
 661:           std::function<SmallVector<Value>(RewriterBase &, Location,
 662:                                            ArrayRef<Value>, Value, int,
 663:                                            VectorType, std::optional<Value>)>
 664:               lowerInst);
```
**EN:** This block declares or defines callable APIs such as lowerLdSt, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 lowerLdSt 等可调用 API，用来封装这里提供的核心行为。

### Lines 666-674
```cpp
 666: // Lower local_load/local_store via ld.shared/st.shared
 667: SmallVector<Value>
 668: lowerLocalLdSt(Location loc, MLIRContext *ctx,
 669:                LinearLayout cvt,          // Map from registers to offset
 670:                ArrayRef<Value> valsArray, // Input for store, empty for load
 671:                Type llvmElemTy, triton::gpu::MemDescType srcTy,
 672:                SharedMemoryObject smemObj, RewriterBase &rewriter,
 673:                const TargetInfoBase &targetInfo,
 674:                Operation *localLoadOp = nullptr);
```
**EN:** This block declares or defines callable APIs such as lowerLocalLdSt, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 lowerLocalLdSt 等可调用 API，用来封装这里提供的核心行为。

### Lines 676-677
```cpp
 676: SmallVector<Value> unpackLLElements(Location loc, Value llvmStruct,
 677:                                     RewriterBase &rewriter);
```
**EN:** This block declares or defines callable APIs such as unpackLLElements, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 unpackLLElements 等可调用 API，用来封装这里提供的核心行为。

### Lines 679-680
```cpp
 679: Value packLLElements(Location loc, const LLVMTypeConverter *typeConverter,
 680:                      ValueRange resultVals, RewriterBase &rewriter, Type type);
```
**EN:** This block declares or defines callable APIs such as packLLElements, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 packLLElements 等可调用 API，用来封装这里提供的核心行为。

### Lines 682-683
```cpp
 682: SmallVector<Value> unpackLLVector(Location loc, Value llvmVec,
 683:                                   RewriterBase &rewriter);
```
**EN:** This block declares or defines callable APIs such as unpackLLVector, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 unpackLLVector 等可调用 API，用来封装这里提供的核心行为。

### Lines 685-685
```cpp
 685: Value packLLVector(Location loc, ValueRange vals, RewriterBase &rewriter);
```
**EN:** This block declares or defines callable APIs such as packLLVector, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 packLLVector 等可调用 API，用来封装这里提供的核心行为。

### Lines 687-687
```cpp
 687: std::optional<LLVM::AtomicBinOp> matchAtomicOp(RMWOp atomicOp);
```
**EN:** This block declares or defines callable APIs such as matchAtomicOp, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 matchAtomicOp 等可调用 API，用来封装这里提供的核心行为。

### Lines 689-689
```cpp
 689: std::optional<LLVM::AtomicOrdering> getMemoryOrdering(MemSemantic memOrdering);
```
**EN:** This block declares or defines callable APIs such as getMemoryOrdering, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getMemoryOrdering 等可调用 API，用来封装这里提供的核心行为。

### Lines 691-691
```cpp
 691: llvm::MapVector<StringAttr, int32_t> getAllFreeVarMasks(MLIRContext *ctx);
```
**EN:** This block declares or defines callable APIs such as getAllFreeVarMasks, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getAllFreeVarMasks 等可调用 API，用来封装这里提供的核心行为。

### Lines 693-693
```cpp
 693: llvm::MapVector<StringAttr, int32_t> getFreeVariableMasks(Type type);
```
**EN:** This block declares or defines callable APIs such as getFreeVariableMasks, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getFreeVariableMasks 等可调用 API，用来封装这里提供的核心行为。

### Lines 695-697
```cpp
 695: inline bool isCanonicalIndex(unsigned index, unsigned freeVarMask) {
 696:   return (index & freeVarMask) == 0;
 697: }
```
**EN:** This block declares or defines callable APIs such as isCanonicalIndex, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isCanonicalIndex 等可调用 API，用来封装这里提供的核心行为。

### Lines 699-701
```cpp
 699: // Certain lowerings may introduce references to function arguments. Keep warp
 700: // group code isolated from above by invoking this function.
 701: void makeAllWarpGroupsIsolatedFromAbove(Operation *op);
```
**EN:** This block declares or defines callable APIs such as makeAllWarpGroupsIsolatedFromAbove, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 makeAllWarpGroupsIsolatedFromAbove 等可调用 API，用来封装这里提供的核心行为。

### Lines 703-704
```cpp
 703: // Set the correct loop annotation on LLVM branch ops.
 704: void fixUpLoopAnnotation(ModuleOp mod);
```
**EN:** This block declares or defines callable APIs such as fixUpLoopAnnotation, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 fixUpLoopAnnotation 等可调用 API，用来封装这里提供的核心行为。

### Lines 706-709
```cpp
 706: void transferSwizzlingLocalMem(triton::gpu::ConvertLayoutOp op, Value src,
 707:                                const TargetInfoBase &targetInfo,
 708:                                const LLVMTypeConverter *typeConverter,
 709:                                RewriterBase &rewriter);
```
**EN:** This block declares or defines callable APIs such as transferSwizzlingLocalMem, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 transferSwizzlingLocalMem 等可调用 API，用来封装这里提供的核心行为。

### Lines 711-714
```cpp
 711: SmallVector<Value> inlineRegionImpl(RewriterBase &rewriter, Region &region,
 712:                                     ArrayRef<Value> args,
 713:                                     mlir::TypeID terminatorTypeId,
 714:                                     Location loc);
```
**EN:** This block declares or defines callable APIs such as inlineRegionImpl, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inlineRegionImpl 等可调用 API，用来封装这里提供的核心行为。

### Lines 716-721
```cpp
 716: template <typename TerminatorOp>
 717: SmallVector<Value> inlineRegion(RewriterBase &rewriter, Region &region,
 718:                                 ArrayRef<Value> args, Location loc) {
 719:   return inlineRegionImpl(rewriter, region, args,
 720:                           mlir::TypeID::get<TerminatorOp>(), loc);
 721: }
```
**EN:** This block declares or defines callable APIs such as inlineRegion and inlineRegionImpl, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inlineRegion and inlineRegionImpl 等可调用 API，用来封装这里提供的核心行为。

### Lines 723-729
```cpp
 723: // #prevBlock
 724: // if (condition) {
 725: //   #ifBlock
 726: // }
 727: // #thenBlock
 728: std::tuple</*prevBlock=*/Block *, /*ifBlock=*/Block *, /*thenBlock=*/Block *>
 729: createIfBlock(RewriterBase &b, Location loc, Value cnd);
```
**EN:** This block declares or defines callable APIs such as createIfBlock, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createIfBlock 等可调用 API，用来封装这里提供的核心行为。

### Lines 731-737
```cpp
 731: void finalizeTensorAtomicResults(Operation *op, RankedTensorType tensorTy,
 732:                                  ConversionPatternRewriter &rewriter,
 733:                                  SmallVector<Value> &resultVals,
 734:                                  Type valueElemTy, TritonLLVMOpBuilder &b,
 735:                                  Value threadPred,
 736:                                  const TargetInfoBase &targetInfo,
 737:                                  const LLVMTypeConverter *typeConverter);
```
**EN:** This block declares or defines callable APIs such as finalizeTensorAtomicResults, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 finalizeTensorAtomicResults 等可调用 API，用来封装这里提供的核心行为。

### Lines 739-748
```cpp
 739: // -----------------------------------------------------------------------
 740: // FuncOp conversion utilities
 741: // -----------------------------------------------------------------------
 742: void filterFuncAttributes(triton::FuncOp op, bool filterArgAttrs,
 743:                           SmallVectorImpl<NamedAttribute> &result);
 744: triton::FuncOp amendFuncOp(triton::FuncOp funcOp,
 745:                            ConversionPatternRewriter &rewriter,
 746:                            const TargetInfoBase &targetInfo);
 747: void handleArgPtrDatatype(triton::FuncOp funcOp, LLVM::LLVMFuncOp &llvmFuncOp);
 748: } // namespace mlir
```
**EN:** This block declares or defines callable APIs such as filterFuncAttributes, amendFuncOp, and handleArgPtrDatatype, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 filterFuncAttributes, amendFuncOp, and handleArgPtrDatatype 等可调用 API，用来封装这里提供的核心行为。

### Lines 750-750
```cpp
 750: #endif
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** linear layout algebra  
  **CN:** 线性布局代数
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** interfaces  
  **CN:** 接口
- **EN:** types  
  **CN:** 类型
- **EN:** swizzling helpers  
  **CN:** swizzle 辅助

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Conversion/LLVMCommon/Pattern.h`
  - `mlir/Dialect/LLVMIR/LLVMDialect.h`
  - `mlir/Interfaces/FunctionInterfaces.h`
  - `triton/Conversion/TritonGPUToLLVM/TargetInfoBase.h`
  - `triton/Dialect/Triton/IR/Dialect.h`
  - `triton/Dialect/Triton/IR/Utility.h`
  - `triton/Dialect/TritonGPU/IR/Dialect.h`
  - `triton/Dialect/TritonGPU/IR/LinearLayoutConversions.h`
  - `triton/Dialect/TritonGPU/IR/Types.h`
  - `triton/Tools/GenericSwizzling.h`
  - `triton/Tools/LinearLayout.h`
  - `triton/Tools/StrUtil.h`
  - `llvm/ADT/STLExtras.h`
- **System or external includes / 系统或外部依赖:**
  - `<optional>`

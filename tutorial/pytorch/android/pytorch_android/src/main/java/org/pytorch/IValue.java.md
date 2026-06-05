# IValue.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/pytorch_android/src/main/java/org/pytorch/IValue.java`
- Repository: `pytorch`
- Purpose (EN): Supports Android/mobile runtime integration in the PyTorch repository.
- 用途 (CN): 为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````java
 1: package org.pytorch;
 2: 
 3: import com.facebook.jni.annotations.DoNotStrip;
 4: import java.util.Locale;
 5: import java.util.Map;
 6: 
 7: /**
 8:  * Java representation of a TorchScript value, which is implemented as tagged union that can be one
 9:  * of the supported types: https://pytorch.org/docs/stable/jit.html#types .
10:  *
11:  * <p>Calling {@code toX} methods for inappropriate types will throw {@link IllegalStateException}.
12:  *
13:  * <p>{@code IValue} objects are constructed with {@code IValue.from(value)}, {@code
14:  * IValue.tupleFrom(value1, value2, ...)}, {@code IValue.listFrom(value1, value2, ...)}, or one of
15:  * the {@code dict} methods, depending on the key type.
16:  *
17:  * <p>Data is retrieved from {@code IValue} objects with the {@code toX()} methods. Note that {@code
18:  * str}-type IValues must be extracted with {@link #toStr()}, rather than {@link #toString()}.
19:  *
20:  * <p>{@code IValue} objects may retain references to objects passed into their constructors, and
21:  * may return references to their internal state from {@code toX()}.
22:  */
23: @DoNotStrip
24: public class IValue {
25:   private static final int TYPE_CODE_NULL = 1;
26: 
27:   private static final int TYPE_CODE_TENSOR = 2;
28:   private static final int TYPE_CODE_BOOL = 3;
29:   private static final int TYPE_CODE_LONG = 4;
30:   private static final int TYPE_CODE_DOUBLE = 5;
31:   private static final int TYPE_CODE_STRING = 6;
32: 
33:   private static final int TYPE_CODE_TUPLE = 7;
34:   private static final int TYPE_CODE_BOOL_LIST = 8;
35:   private static final int TYPE_CODE_LONG_LIST = 9;
36:   private static final int TYPE_CODE_DOUBLE_LIST = 10;
37:   private static final int TYPE_CODE_TENSOR_LIST = 11;
38:   private static final int TYPE_CODE_LIST = 12;
39: 
40:   private static final int TYPE_CODE_DICT_STRING_KEY = 13;
41:   private static final int TYPE_CODE_DICT_LONG_KEY = 14;
42: 
43:   private String[] TYPE_NAMES = {
44:     "Unknown",
45:     "Null",
46:     "Tensor",
47:     "Bool",
48:     "Long",
49:     "Double",
50:     "String",
51:     "Tuple",
52:     "BoolList",
53:     "LongList",
54:     "DoubleList",
55:     "TensorList",
56:     "GenericList",
57:     "DictStringKey",
58:     "DictLongKey",
59:   };
60: 
````
- EN: Handles module imports such as `com.facebook.jni.annotations.DoNotStrip;`, `java.util.Locale;`, `java.util.Map;`.
- CN: 处理模块导入，例如 `com.facebook.jni.annotations.DoNotStrip;`, `java.util.Locale;`, `java.util.Map;`。
- EN: Imports Java types such as `com.facebook.jni.annotations.DoNotStrip`, `java.util.Locale`, `java.util.Map`.
- CN: 导入 Java 类型，例如 `com.facebook.jni.annotations.DoNotStrip`, `java.util.Locale`, `java.util.Map`。
- EN: Declares or extends types including `IValue`.
- CN: 声明或扩展类型，包括 `IValue`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````java
 61:   @DoNotStrip private final int mTypeCode;
 62:   @DoNotStrip private Object mData;
 63: 
 64:   @DoNotStrip
 65:   private IValue(int typeCode) {
 66:     this.mTypeCode = typeCode;
 67:   }
 68: 
 69:   @DoNotStrip
 70:   public boolean isNull() {
 71:     return TYPE_CODE_NULL == this.mTypeCode;
 72:   }
 73: 
 74:   @DoNotStrip
 75:   public boolean isTensor() {
 76:     return TYPE_CODE_TENSOR == this.mTypeCode;
 77:   }
 78: 
 79:   @DoNotStrip
 80:   public boolean isBool() {
 81:     return TYPE_CODE_BOOL == this.mTypeCode;
 82:   }
 83: 
 84:   @DoNotStrip
 85:   public boolean isLong() {
 86:     return TYPE_CODE_LONG == this.mTypeCode;
 87:   }
 88: 
 89:   @DoNotStrip
 90:   public boolean isDouble() {
 91:     return TYPE_CODE_DOUBLE == this.mTypeCode;
 92:   }
 93: 
 94:   @DoNotStrip
 95:   public boolean isString() {
 96:     return TYPE_CODE_STRING == this.mTypeCode;
 97:   }
 98: 
 99:   @DoNotStrip
100:   public boolean isTuple() {
101:     return TYPE_CODE_TUPLE == this.mTypeCode;
102:   }
103: 
104:   @DoNotStrip
105:   public boolean isBoolList() {
106:     return TYPE_CODE_BOOL_LIST == this.mTypeCode;
107:   }
108: 
109:   @DoNotStrip
110:   public boolean isLongList() {
111:     return TYPE_CODE_LONG_LIST == this.mTypeCode;
112:   }
113: 
114:   @DoNotStrip
115:   public boolean isDoubleList() {
116:     return TYPE_CODE_DOUBLE_LIST == this.mTypeCode;
117:   }
118: 
119:   @DoNotStrip
120:   public boolean isTensorList() {
````
- EN: Implements callable logic such as `IValue`, `isNull`, `isTensor`, `isBool`.
- CN: 实现可调用逻辑，例如 `IValue`, `isNull`, `isTensor`, `isBool`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````java
121:     return TYPE_CODE_TENSOR_LIST == this.mTypeCode;
122:   }
123: 
124:   @DoNotStrip
125:   public boolean isList() {
126:     return TYPE_CODE_LIST == this.mTypeCode;
127:   }
128: 
129:   @DoNotStrip
130:   public boolean isDictStringKey() {
131:     return TYPE_CODE_DICT_STRING_KEY == this.mTypeCode;
132:   }
133: 
134:   @DoNotStrip
135:   public boolean isDictLongKey() {
136:     return TYPE_CODE_DICT_LONG_KEY == this.mTypeCode;
137:   }
138: 
139:   /** Creates a new {@code IValue} of type {@code Optional} that contains no value. */
140:   @DoNotStrip
141:   public static IValue optionalNull() {
142:     return new IValue(TYPE_CODE_NULL);
143:   }
144:   /** Creates a new {@code IValue} of type {@code Tensor}. */
145:   @DoNotStrip
146:   public static IValue from(Tensor tensor) {
147:     final IValue iv = new IValue(TYPE_CODE_TENSOR);
148:     iv.mData = tensor;
149:     return iv;
150:   }
151:   /** Creates a new {@code IValue} of type {@code bool}. */
152:   @DoNotStrip
153:   public static IValue from(boolean value) {
154:     final IValue iv = new IValue(TYPE_CODE_BOOL);
155:     iv.mData = value;
156:     return iv;
157:   }
158: 
159:   /** Creates a new {@code IValue} of type {@code int}. */
160:   @DoNotStrip
161:   public static IValue from(long value) {
162:     final IValue iv = new IValue(TYPE_CODE_LONG);
163:     iv.mData = value;
164:     return iv;
165:   }
166:   /** Creates a new {@code IValue} of type {@code float}. */
167:   @DoNotStrip
168:   public static IValue from(double value) {
169:     final IValue iv = new IValue(TYPE_CODE_DOUBLE);
170:     iv.mData = value;
171:     return iv;
172:   }
173:   /** Creates a new {@code IValue} of type {@code str}. */
174:   @DoNotStrip
175:   public static IValue from(String value) {
176:     final IValue iv = new IValue(TYPE_CODE_STRING);
177:     iv.mData = value;
178:     return iv;
179:   }
180: 
````
- EN: Implements callable logic such as `isList`, `isDictStringKey`, `isDictLongKey`, `optionalNull`.
- CN: 实现可调用逻辑，例如 `isList`, `isDictStringKey`, `isDictLongKey`, `optionalNull`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````java
181:   /** Creates a new {@code IValue} of type {@code List[bool]}. */
182:   @DoNotStrip
183:   public static IValue listFrom(boolean... list) {
184:     final IValue iv = new IValue(TYPE_CODE_BOOL_LIST);
185:     iv.mData = list;
186:     return iv;
187:   }
188:   /** Creates a new {@code IValue} of type {@code List[int]}. */
189:   @DoNotStrip
190:   public static IValue listFrom(long... list) {
191:     final IValue iv = new IValue(TYPE_CODE_LONG_LIST);
192:     iv.mData = list;
193:     return iv;
194:   }
195:   /** Creates a new {@code IValue} of type {@code List[float]}. */
196:   @DoNotStrip
197:   public static IValue listFrom(double... list) {
198:     final IValue iv = new IValue(TYPE_CODE_DOUBLE_LIST);
199:     iv.mData = list;
200:     return iv;
201:   }
202: 
203:   /** Creates a new {@code IValue} of type {@code List[Tensor]}. */
204:   @DoNotStrip
205:   public static IValue listFrom(Tensor... list) {
206:     final IValue iv = new IValue(TYPE_CODE_TENSOR_LIST);
207:     iv.mData = list;
208:     return iv;
209:   }
210: 
211:   /** Creates a new {@code IValue} of type {@code List[T]}. All elements must have the same type. */
212:   @DoNotStrip
213:   public static IValue listFrom(IValue... array) {
214:     final int size = array.length;
215:     if (size > 0) {
216:       final int typeCode0 = array[0].mTypeCode;
217:       for (int i = 1; i < size; i++) {
218:         if (typeCode0 != array[i].mTypeCode) {
219:           throw new IllegalArgumentException("List must contain items of the same type");
220:         }
221:       }
222:     }
223: 
224:     final IValue iv = new IValue(TYPE_CODE_LIST);
225:     iv.mData = array;
226:     return iv;
227:   }
228:   /** Creates a new {@code IValue} of type {@code Tuple[T0, T1, ...]}. */
229:   @DoNotStrip
230:   public static IValue tupleFrom(IValue... array) {
231:     final IValue iv = new IValue(TYPE_CODE_TUPLE);
232:     iv.mData = array;
233:     return iv;
234:   }
235: 
236:   /** Creates a new {@code IValue} of type {@code Dict[str, V]}. */
237:   @DoNotStrip
238:   public static IValue dictStringKeyFrom(Map<String, IValue> map) {
239:     final IValue iv = new IValue(TYPE_CODE_DICT_STRING_KEY);
240:     iv.mData = map;
````
- EN: Implements callable logic such as `listFrom`, `tupleFrom`, `dictStringKeyFrom`.
- CN: 实现可调用逻辑，例如 `listFrom`, `tupleFrom`, `dictStringKeyFrom`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````java
241:     return iv;
242:   }
243:   /** Creates a new {@code IValue} of type {@code Dict[int, V]}. */
244:   @DoNotStrip
245:   public static IValue dictLongKeyFrom(Map<Long, IValue> map) {
246:     final IValue iv = new IValue(TYPE_CODE_DICT_LONG_KEY);
247:     iv.mData = map;
248:     return iv;
249:   }
250: 
251:   @DoNotStrip
252:   public Tensor toTensor() {
253:     preconditionType(TYPE_CODE_TENSOR, mTypeCode);
254:     return (Tensor) mData;
255:   }
256: 
257:   @DoNotStrip
258:   public boolean toBool() {
259:     preconditionType(TYPE_CODE_BOOL, mTypeCode);
260:     return (boolean) mData;
261:   }
262: 
263:   @DoNotStrip
264:   public long toLong() {
265:     preconditionType(TYPE_CODE_LONG, mTypeCode);
266:     return (long) mData;
267:   }
268: 
269:   @DoNotStrip
270:   public double toDouble() {
271:     preconditionType(TYPE_CODE_DOUBLE, mTypeCode);
272:     return (double) mData;
273:   }
274: 
275:   @DoNotStrip
276:   public String toStr() {
277:     preconditionType(TYPE_CODE_STRING, mTypeCode);
278:     return (String) mData;
279:   }
280: 
281:   @DoNotStrip
282:   public boolean[] toBoolList() {
283:     preconditionType(TYPE_CODE_BOOL_LIST, mTypeCode);
284:     return (boolean[]) mData;
285:   }
286: 
287:   @DoNotStrip
288:   public long[] toLongList() {
289:     preconditionType(TYPE_CODE_LONG_LIST, mTypeCode);
290:     return (long[]) mData;
291:   }
292: 
293:   @DoNotStrip
294:   public double[] toDoubleList() {
295:     preconditionType(TYPE_CODE_DOUBLE_LIST, mTypeCode);
296:     return (double[]) mData;
297:   }
298: 
299:   @DoNotStrip
300:   public Tensor[] toTensorList() {
````
- EN: Implements callable logic such as `dictLongKeyFrom`, `toTensor`, `toBool`, `toLong`.
- CN: 实现可调用逻辑，例如 `dictLongKeyFrom`, `toTensor`, `toBool`, `toLong`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-343
````java
301:     preconditionType(TYPE_CODE_TENSOR_LIST, mTypeCode);
302:     return (Tensor[]) mData;
303:   }
304: 
305:   @DoNotStrip
306:   public IValue[] toList() {
307:     preconditionType(TYPE_CODE_LIST, mTypeCode);
308:     return (IValue[]) mData;
309:   }
310: 
311:   @DoNotStrip
312:   public IValue[] toTuple() {
313:     preconditionType(TYPE_CODE_TUPLE, mTypeCode);
314:     return (IValue[]) mData;
315:   }
316: 
317:   @DoNotStrip
318:   public Map<String, IValue> toDictStringKey() {
319:     preconditionType(TYPE_CODE_DICT_STRING_KEY, mTypeCode);
320:     return (Map<String, IValue>) mData;
321:   }
322: 
323:   @DoNotStrip
324:   public Map<Long, IValue> toDictLongKey() {
325:     preconditionType(TYPE_CODE_DICT_LONG_KEY, mTypeCode);
326:     return (Map<Long, IValue>) mData;
327:   }
328: 
329:   private void preconditionType(int typeCodeExpected, int typeCode) {
330:     if (typeCode != typeCodeExpected) {
331:       throw new IllegalStateException(
332:           String.format(
333:               Locale.US,
334:               "Expected IValue type %s, actual type %s",
335:               getTypeName(typeCodeExpected),
336:               getTypeName(typeCode)));
337:     }
338:   }
339: 
340:   private String getTypeName(int typeCode) {
341:     return typeCode >= 0 && typeCode < TYPE_NAMES.length ? TYPE_NAMES[typeCode] : "Unknown";
342:   }
343: }
````
- EN: Implements callable logic such as `toList`, `toTuple`, `preconditionType`, `getTypeName`.
- CN: 实现可调用逻辑，例如 `toList`, `toTuple`, `preconditionType`, `getTypeName`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Symbol `IValue` / 符号 `IValue`
- Symbol `isNull` / 符号 `isNull`
- Symbol `isTensor` / 符号 `isTensor`
- Symbol `isBool` / 符号 `isBool`

## Dependencies / 依赖关系
- Java imports: `com.facebook.jni.annotations.DoNotStrip`, `java.util.Locale`, `java.util.Map`
- Java 导入: `com.facebook.jni.annotations.DoNotStrip`, `java.util.Locale`, `java.util.Map`

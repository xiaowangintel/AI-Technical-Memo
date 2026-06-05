# TensorImageUtils.java — Code Analysis / 代码分析

## Source / 来源
- File: `android/pytorch_android_torchvision/src/main/java/org/pytorch/torchvision/TensorImageUtils.java`
- Repository: `pytorch`
- Purpose (EN): Supports Android/mobile runtime integration in the PyTorch repository.
- 用途 (CN): 为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````java
 1: package org.pytorch.torchvision;
 2: 
 3: import android.graphics.Bitmap;
 4: import android.graphics.ImageFormat;
 5: import android.media.Image;
 6: import com.facebook.soloader.nativeloader.NativeLoader;
 7: import com.facebook.soloader.nativeloader.SystemDelegate;
 8: import java.nio.Buffer;
 9: import java.nio.ByteBuffer;
10: import java.nio.FloatBuffer;
11: import java.util.Locale;
12: import org.pytorch.MemoryFormat;
13: import org.pytorch.Tensor;
14: 
15: /**
16:  * Contains utility functions for {@link org.pytorch.Tensor} creation from {@link
17:  * android.graphics.Bitmap} or {@link android.media.Image} source.
18:  */
19: public final class TensorImageUtils {
20: 
21:   public static float[] TORCHVISION_NORM_MEAN_RGB = new float[] {0.485f, 0.456f, 0.406f};
22:   public static float[] TORCHVISION_NORM_STD_RGB = new float[] {0.229f, 0.224f, 0.225f};
23: 
24:   /**
25:    * Creates new {@link org.pytorch.Tensor} from full {@link android.graphics.Bitmap}, normalized
26:    * with specified in parameters mean and std.
27:    *
28:    * @param normMeanRGB means for RGB channels normalization, length must equal 3, RGB order
29:    * @param normStdRGB standard deviation for RGB channels normalization, length must equal 3, RGB
30:    *     order
31:    */
32:   public static Tensor bitmapToFloat32Tensor(
33:       final Bitmap bitmap,
34:       final float[] normMeanRGB,
35:       final float normStdRGB[],
36:       final MemoryFormat memoryFormat) {
37:     checkNormMeanArg(normMeanRGB);
38:     checkNormStdArg(normStdRGB);
39: 
40:     return bitmapToFloat32Tensor(
41:         bitmap, 0, 0, bitmap.getWidth(), bitmap.getHeight(), normMeanRGB, normStdRGB, memoryFormat);
42:   }
43: 
44:   public static Tensor bitmapToFloat32Tensor(
45:       final Bitmap bitmap, final float[] normMeanRGB, final float normStdRGB[]) {
46:     return bitmapToFloat32Tensor(
47:         bitmap,
48:         0,
49:         0,
50:         bitmap.getWidth(),
51:         bitmap.getHeight(),
52:         normMeanRGB,
53:         normStdRGB,
54:         MemoryFormat.CONTIGUOUS);
55:   }
56: 
57:   /**
58:    * Writes tensor content from specified {@link android.graphics.Bitmap}, normalized with specified
59:    * in parameters mean and std to specified {@link java.nio.FloatBuffer} with specified offset.
60:    *
````
- EN: Handles module imports such as `android.graphics.Bitmap;`, `android.graphics.ImageFormat;`, `android.media.Image;`, `com.facebook.soloader.nativeloader.NativeLoader;`.
- CN: 处理模块导入，例如 `android.graphics.Bitmap;`, `android.graphics.ImageFormat;`, `android.media.Image;`, `com.facebook.soloader.nativeloader.NativeLoader;`。
- EN: Imports Java types such as `android.graphics.Bitmap`, `android.graphics.ImageFormat`, `android.media.Image`, `com.facebook.soloader.nativeloader.NativeLoader`.
- CN: 导入 Java 类型，例如 `android.graphics.Bitmap`, `android.graphics.ImageFormat`, `android.media.Image`, `com.facebook.soloader.nativeloader.NativeLoader`。
- EN: Declares or extends types including `TensorImageUtils`.
- CN: 声明或扩展类型，包括 `TensorImageUtils`。
- EN: Implements callable logic such as `bitmapToFloat32Tensor`.
- CN: 实现可调用逻辑，例如 `bitmapToFloat32Tensor`。

### Lines 61-120
````java
 61:    * @param bitmap {@link android.graphics.Bitmap} as a source for Tensor data
 62:    * @param x - x coordinate of top left corner of bitmap's area
 63:    * @param y - y coordinate of top left corner of bitmap's area
 64:    * @param width - width of bitmap's area
 65:    * @param height - height of bitmap's area
 66:    * @param normMeanRGB means for RGB channels normalization, length must equal 3, RGB order
 67:    * @param normStdRGB standard deviation for RGB channels normalization, length must equal 3, RGB
 68:    *     order
 69:    */
 70:   public static void bitmapToFloatBuffer(
 71:       final Bitmap bitmap,
 72:       final int x,
 73:       final int y,
 74:       final int width,
 75:       final int height,
 76:       final float[] normMeanRGB,
 77:       final float[] normStdRGB,
 78:       final FloatBuffer outBuffer,
 79:       final int outBufferOffset,
 80:       final MemoryFormat memoryFormat) {
 81:     checkOutBufferCapacity(outBuffer, outBufferOffset, width, height);
 82:     checkNormMeanArg(normMeanRGB);
 83:     checkNormStdArg(normStdRGB);
 84:     if (memoryFormat != MemoryFormat.CONTIGUOUS && memoryFormat != MemoryFormat.CHANNELS_LAST) {
 85:       throw new IllegalArgumentException("Unsupported memory format " + memoryFormat);
 86:     }
 87: 
 88:     final int pixelsCount = height * width;
 89:     final int[] pixels = new int[pixelsCount];
 90:     bitmap.getPixels(pixels, 0, width, x, y, width, height);
 91:     if (MemoryFormat.CONTIGUOUS == memoryFormat) {
 92:       final int offset_g = pixelsCount;
 93:       final int offset_b = 2 * pixelsCount;
 94:       for (int i = 0; i < pixelsCount; i++) {
 95:         final int c = pixels[i];
 96:         float r = ((c >> 16) & 0xff) / 255.0f;
 97:         float g = ((c >> 8) & 0xff) / 255.0f;
 98:         float b = ((c) & 0xff) / 255.0f;
 99:         outBuffer.put(outBufferOffset + i, (r - normMeanRGB[0]) / normStdRGB[0]);
100:         outBuffer.put(outBufferOffset + offset_g + i, (g - normMeanRGB[1]) / normStdRGB[1]);
101:         outBuffer.put(outBufferOffset + offset_b + i, (b - normMeanRGB[2]) / normStdRGB[2]);
102:       }
103:     } else {
104:       for (int i = 0; i < pixelsCount; i++) {
105:         final int c = pixels[i];
106:         float r = ((c >> 16) & 0xff) / 255.0f;
107:         float g = ((c >> 8) & 0xff) / 255.0f;
108:         float b = ((c) & 0xff) / 255.0f;
109:         outBuffer.put(outBufferOffset + 3 * i + 0, (r - normMeanRGB[0]) / normStdRGB[0]);
110:         outBuffer.put(outBufferOffset + 3 * i + 1, (g - normMeanRGB[1]) / normStdRGB[1]);
111:         outBuffer.put(outBufferOffset + 3 * i + 2, (b - normMeanRGB[2]) / normStdRGB[2]);
112:       }
113:     }
114:   }
115: 
116:   public static void bitmapToFloatBuffer(
117:       final Bitmap bitmap,
118:       final int x,
119:       final int y,
120:       final int width,
````
- EN: Implements callable logic such as `bitmapToFloatBuffer`.
- CN: 实现可调用逻辑，例如 `bitmapToFloatBuffer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-180
````java
121:       final int height,
122:       final float[] normMeanRGB,
123:       final float[] normStdRGB,
124:       final FloatBuffer outBuffer,
125:       final int outBufferOffset) {
126:     bitmapToFloatBuffer(
127:         bitmap,
128:         x,
129:         y,
130:         width,
131:         height,
132:         normMeanRGB,
133:         normStdRGB,
134:         outBuffer,
135:         outBufferOffset,
136:         MemoryFormat.CONTIGUOUS);
137:   }
138: 
139:   /**
140:    * Creates new {@link org.pytorch.Tensor} from specified area of {@link android.graphics.Bitmap},
141:    * normalized with specified in parameters mean and std.
142:    *
143:    * @param bitmap {@link android.graphics.Bitmap} as a source for Tensor data
144:    * @param x - x coordinate of top left corner of bitmap's area
145:    * @param y - y coordinate of top left corner of bitmap's area
146:    * @param width - width of bitmap's area
147:    * @param height - height of bitmap's area
148:    * @param normMeanRGB means for RGB channels normalization, length must equal 3, RGB order
149:    * @param normStdRGB standard deviation for RGB channels normalization, length must equal 3, RGB
150:    *     order
151:    */
152:   public static Tensor bitmapToFloat32Tensor(
153:       final Bitmap bitmap,
154:       int x,
155:       int y,
156:       int width,
157:       int height,
158:       float[] normMeanRGB,
159:       float[] normStdRGB,
160:       MemoryFormat memoryFormat) {
161:     checkNormMeanArg(normMeanRGB);
162:     checkNormStdArg(normStdRGB);
163: 
164:     final FloatBuffer floatBuffer = Tensor.allocateFloatBuffer(3 * width * height);
165:     bitmapToFloatBuffer(
166:         bitmap, x, y, width, height, normMeanRGB, normStdRGB, floatBuffer, 0, memoryFormat);
167:     return Tensor.fromBlob(floatBuffer, new long[] {1, 3, height, width}, memoryFormat);
168:   }
169: 
170:   public static Tensor bitmapToFloat32Tensor(
171:       final Bitmap bitmap,
172:       int x,
173:       int y,
174:       int width,
175:       int height,
176:       float[] normMeanRGB,
177:       float[] normStdRGB) {
178:     return bitmapToFloat32Tensor(
179:         bitmap, x, y, width, height, normMeanRGB, normStdRGB, MemoryFormat.CONTIGUOUS);
180:   }
````
- EN: Implements callable logic such as `bitmapToFloat32Tensor`.
- CN: 实现可调用逻辑，例如 `bitmapToFloat32Tensor`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````java
181: 
182:   /**
183:    * Creates new {@link org.pytorch.Tensor} from specified area of {@link android.media.Image},
184:    * doing optional rotation, scaling (nearest) and center cropping.
185:    *
186:    * @param image {@link android.media.Image} as a source for Tensor data
187:    * @param rotateCWDegrees Clockwise angle through which the input image needs to be rotated to be
188:    *     upright. Range of valid values: 0, 90, 180, 270
189:    * @param tensorWidth return tensor width, must be positive
190:    * @param tensorHeight return tensor height, must be positive
191:    * @param normMeanRGB means for RGB channels normalization, length must equal 3, RGB order
192:    * @param normStdRGB standard deviation for RGB channels normalization, length must equal 3, RGB
193:    *     order
194:    */
195:   public static Tensor imageYUV420CenterCropToFloat32Tensor(
196:       final Image image,
197:       int rotateCWDegrees,
198:       final int tensorWidth,
199:       final int tensorHeight,
200:       float[] normMeanRGB,
201:       float[] normStdRGB,
202:       MemoryFormat memoryFormat) {
203:     if (image.getFormat() != ImageFormat.YUV_420_888) {
204:       throw new IllegalArgumentException(
205:           String.format(
206:               Locale.US, "Image format %d != ImageFormat.YUV_420_888", image.getFormat()));
207:     }
208: 
209:     checkNormMeanArg(normMeanRGB);
210:     checkNormStdArg(normStdRGB);
211:     checkRotateCWDegrees(rotateCWDegrees);
212:     checkTensorSize(tensorWidth, tensorHeight);
213: 
214:     final FloatBuffer floatBuffer = Tensor.allocateFloatBuffer(3 * tensorWidth * tensorHeight);
215:     imageYUV420CenterCropToFloatBuffer(
216:         image,
217:         rotateCWDegrees,
218:         tensorWidth,
219:         tensorHeight,
220:         normMeanRGB,
221:         normStdRGB,
222:         floatBuffer,
223:         0,
224:         memoryFormat);
225:     return Tensor.fromBlob(floatBuffer, new long[] {1, 3, tensorHeight, tensorWidth}, memoryFormat);
226:   }
227: 
228:   public static Tensor imageYUV420CenterCropToFloat32Tensor(
229:       final Image image,
230:       int rotateCWDegrees,
231:       final int tensorWidth,
232:       final int tensorHeight,
233:       float[] normMeanRGB,
234:       float[] normStdRGB) {
235:     return imageYUV420CenterCropToFloat32Tensor(
236:         image,
237:         rotateCWDegrees,
238:         tensorWidth,
239:         tensorHeight,
240:         normMeanRGB,
````
- EN: Implements callable logic such as `imageYUV420CenterCropToFloat32Tensor`.
- CN: 实现可调用逻辑，例如 `imageYUV420CenterCropToFloat32Tensor`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````java
241:         normStdRGB,
242:         MemoryFormat.CONTIGUOUS);
243:   }
244: 
245:   /**
246:    * Writes tensor content from specified {@link android.media.Image}, doing optional rotation,
247:    * scaling (nearest) and center cropping to specified {@link java.nio.FloatBuffer} with specified
248:    * offset.
249:    *
250:    * @param image {@link android.media.Image} as a source for Tensor data
251:    * @param rotateCWDegrees Clockwise angle through which the input image needs to be rotated to be
252:    *     upright. Range of valid values: 0, 90, 180, 270
253:    * @param tensorWidth return tensor width, must be positive
254:    * @param tensorHeight return tensor height, must be positive
255:    * @param normMeanRGB means for RGB channels normalization, length must equal 3, RGB order
256:    * @param normStdRGB standard deviation for RGB channels normalization, length must equal 3, RGB
257:    *     order
258:    * @param outBuffer Output buffer, where tensor content will be written
259:    * @param outBufferOffset Output buffer offset with which tensor content will be written
260:    */
261:   public static void imageYUV420CenterCropToFloatBuffer(
262:       final Image image,
263:       int rotateCWDegrees,
264:       final int tensorWidth,
265:       final int tensorHeight,
266:       float[] normMeanRGB,
267:       float[] normStdRGB,
268:       final FloatBuffer outBuffer,
269:       final int outBufferOffset,
270:       final MemoryFormat memoryFormat) {
271:     checkOutBufferCapacity(outBuffer, outBufferOffset, tensorWidth, tensorHeight);
272: 
273:     if (image.getFormat() != ImageFormat.YUV_420_888) {
274:       throw new IllegalArgumentException(
275:           String.format(
276:               Locale.US, "Image format %d != ImageFormat.YUV_420_888", image.getFormat()));
277:     }
278: 
279:     checkNormMeanArg(normMeanRGB);
280:     checkNormStdArg(normStdRGB);
281:     checkRotateCWDegrees(rotateCWDegrees);
282:     checkTensorSize(tensorWidth, tensorHeight);
283: 
284:     Image.Plane[] planes = image.getPlanes();
285:     Image.Plane Y = planes[0];
286:     Image.Plane U = planes[1];
287:     Image.Plane V = planes[2];
288: 
289:     int memoryFormatJniCode = 0;
290:     if (MemoryFormat.CONTIGUOUS == memoryFormat) {
291:       memoryFormatJniCode = 1;
292:     } else if (MemoryFormat.CHANNELS_LAST == memoryFormat) {
293:       memoryFormatJniCode = 2;
294:     }
295: 
296:     NativePeer.imageYUV420CenterCropToFloatBuffer(
297:         Y.getBuffer(),
298:         Y.getRowStride(),
299:         Y.getPixelStride(),
300:         U.getBuffer(),
````
- EN: Implements callable logic such as `imageYUV420CenterCropToFloatBuffer`.
- CN: 实现可调用逻辑，例如 `imageYUV420CenterCropToFloatBuffer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-360
````java
301:         V.getBuffer(),
302:         U.getRowStride(),
303:         U.getPixelStride(),
304:         image.getWidth(),
305:         image.getHeight(),
306:         rotateCWDegrees,
307:         tensorWidth,
308:         tensorHeight,
309:         normMeanRGB,
310:         normStdRGB,
311:         outBuffer,
312:         outBufferOffset,
313:         memoryFormatJniCode);
314:   }
315: 
316:   public static void imageYUV420CenterCropToFloatBuffer(
317:       final Image image,
318:       int rotateCWDegrees,
319:       final int tensorWidth,
320:       final int tensorHeight,
321:       float[] normMeanRGB,
322:       float[] normStdRGB,
323:       final FloatBuffer outBuffer,
324:       final int outBufferOffset) {
325:     imageYUV420CenterCropToFloatBuffer(
326:         image,
327:         rotateCWDegrees,
328:         tensorWidth,
329:         tensorHeight,
330:         normMeanRGB,
331:         normStdRGB,
332:         outBuffer,
333:         outBufferOffset,
334:         MemoryFormat.CONTIGUOUS);
335:   }
336: 
337:   private static class NativePeer {
338:     static {
339:       if (!NativeLoader.isInitialized()) {
340:         NativeLoader.init(new SystemDelegate());
341:       }
342:       NativeLoader.loadLibrary("pytorch_vision_jni");
343:     }
344: 
345:     private static native void imageYUV420CenterCropToFloatBuffer(
346:         ByteBuffer yBuffer,
347:         int yRowStride,
348:         int yPixelStride,
349:         ByteBuffer uBuffer,
350:         ByteBuffer vBuffer,
351:         int uvRowStride,
352:         int uvPixelStride,
353:         int imageWidth,
354:         int imageHeight,
355:         int rotateCWDegrees,
356:         int tensorWidth,
357:         int tensorHeight,
358:         float[] normMeanRgb,
359:         float[] normStdRgb,
360:         Buffer outBuffer,
````
- EN: Declares or extends types including `NativePeer`.
- CN: 声明或扩展类型，包括 `NativePeer`。
- EN: Implements callable logic such as `imageYUV420CenterCropToFloatBuffer`.
- CN: 实现可调用逻辑，例如 `imageYUV420CenterCropToFloatBuffer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 361-398
````java
361:         int outBufferOffset,
362:         int memoryFormatJniCode);
363:   }
364: 
365:   private static void checkOutBufferCapacity(
366:       FloatBuffer outBuffer, int outBufferOffset, int tensorWidth, int tensorHeight) {
367:     if (outBufferOffset + 3 * tensorWidth * tensorHeight > outBuffer.capacity()) {
368:       throw new IllegalStateException("Buffer underflow");
369:     }
370:   }
371: 
372:   private static void checkTensorSize(int tensorWidth, int tensorHeight) {
373:     if (tensorHeight <= 0 || tensorWidth <= 0) {
374:       throw new IllegalArgumentException("tensorHeight and tensorWidth must be positive");
375:     }
376:   }
377: 
378:   private static void checkRotateCWDegrees(int rotateCWDegrees) {
379:     if (rotateCWDegrees != 0
380:         && rotateCWDegrees != 90
381:         && rotateCWDegrees != 180
382:         && rotateCWDegrees != 270) {
383:       throw new IllegalArgumentException("rotateCWDegrees must be one of 0, 90, 180, 270");
384:     }
385:   }
386: 
387:   private static void checkNormStdArg(float[] normStdRGB) {
388:     if (normStdRGB.length != 3) {
389:       throw new IllegalArgumentException("normStdRGB length must be 3");
390:     }
391:   }
392: 
393:   private static void checkNormMeanArg(float[] normMeanRGB) {
394:     if (normMeanRGB.length != 3) {
395:       throw new IllegalArgumentException("normMeanRGB length must be 3");
396:     }
397:   }
398: }
````
- EN: Implements callable logic such as `checkOutBufferCapacity`, `checkTensorSize`, `checkRotateCWDegrees`, `checkNormStdArg`.
- CN: 实现可调用逻辑，例如 `checkOutBufferCapacity`, `checkTensorSize`, `checkRotateCWDegrees`, `checkNormStdArg`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Symbol `TensorImageUtils` / 符号 `TensorImageUtils`
- Symbol `NativePeer` / 符号 `NativePeer`
- Symbol `bitmapToFloat32Tensor` / 符号 `bitmapToFloat32Tensor`
- Symbol `bitmapToFloatBuffer` / 符号 `bitmapToFloatBuffer`

## Dependencies / 依赖关系
- Java imports: `android.graphics.Bitmap`, `android.graphics.ImageFormat`, `android.media.Image`, `com.facebook.soloader.nativeloader.NativeLoader`, `com.facebook.soloader.nativeloader.SystemDelegate`, `java.nio.Buffer`, `java.nio.ByteBuffer`, `java.nio.FloatBuffer`, `java.util.Locale`, `org.pytorch.MemoryFormat`
- Java 导入: `android.graphics.Bitmap`, `android.graphics.ImageFormat`, `android.media.Image`, `com.facebook.soloader.nativeloader.NativeLoader`, `com.facebook.soloader.nativeloader.SystemDelegate`, `java.nio.Buffer`, `java.nio.ByteBuffer`, `java.nio.FloatBuffer`, `java.util.Locale`, `org.pytorch.MemoryFormat`

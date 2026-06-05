# pytorch_vision_jni.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/pytorch_android_torchvision/src/main/cpp/pytorch_vision_jni.cpp`
- Repository: `pytorch`
- Purpose (EN): Supports Android/mobile runtime integration in the PyTorch repository.
- 用途 (CN): 为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````cpp
 1: #include <cassert>
 2: #include <cmath>
 3: #include <vector>
 4: 
 5: #include "jni.h"
 6: 
 7: namespace pytorch_vision_jni {
 8: 
 9: static void imageYUV420CenterCropToFloatBuffer(
10:     JNIEnv* jniEnv,
11:     jclass,
12:     jobject yBuffer,
13:     jint yRowStride,
14:     jint yPixelStride,
15:     jobject uBuffer,
16:     jobject vBuffer,
17:     jint uRowStride,
18:     jint uvPixelStride,
19:     jint imageWidth,
20:     jint imageHeight,
21:     jint rotateCWDegrees,
22:     jint tensorWidth,
23:     jint tensorHeight,
24:     jfloatArray jnormMeanRGB,
25:     jfloatArray jnormStdRGB,
26:     jobject outBuffer,
27:     jint outOffset,
28:     jint memoryFormatCode) {
29:   constexpr static int32_t kMemoryFormatContiguous = 1;
30:   constexpr static int32_t kMemoryFormatChannelsLast = 2;
31: 
32:   float* outData = (float*)jniEnv->GetDirectBufferAddress(outBuffer);
33: 
34:   jfloat normMeanRGB[3];
35:   jfloat normStdRGB[3];
36:   jniEnv->GetFloatArrayRegion(jnormMeanRGB, 0, 3, normMeanRGB);
37:   jniEnv->GetFloatArrayRegion(jnormStdRGB, 0, 3, normStdRGB);
38:   int widthAfterRtn = imageWidth;
39:   int heightAfterRtn = imageHeight;
40:   bool oddRotation = rotateCWDegrees == 90 || rotateCWDegrees == 270;
````
- EN: Pulls in native headers such as `cassert`, `cmath`, `vector`, `jni.h`.
- CN: 引入原生头文件，例如 `cassert`, `cmath`, `vector`, `jni.h`。
- EN: Implements callable logic such as `imageYUV420CenterCropToFloatBuffer`.
- CN: 实现可调用逻辑，例如 `imageYUV420CenterCropToFloatBuffer`。

### Lines 41-80
````cpp
41:   if (oddRotation) {
42:     widthAfterRtn = imageHeight;
43:     heightAfterRtn = imageWidth;
44:   }
45: 
46:   int cropWidthAfterRtn = widthAfterRtn;
47:   int cropHeightAfterRtn = heightAfterRtn;
48: 
49:   if (tensorWidth * heightAfterRtn <= tensorHeight * widthAfterRtn) {
50:     cropWidthAfterRtn = tensorWidth * heightAfterRtn / tensorHeight;
51:   } else {
52:     cropHeightAfterRtn = tensorHeight * widthAfterRtn / tensorWidth;
53:   }
54: 
55:   int cropWidthBeforeRtn = cropWidthAfterRtn;
56:   int cropHeightBeforeRtn = cropHeightAfterRtn;
57:   if (oddRotation) {
58:     cropWidthBeforeRtn = cropHeightAfterRtn;
59:     cropHeightBeforeRtn = cropWidthAfterRtn;
60:   }
61: 
62:   const int offsetX = (imageWidth - cropWidthBeforeRtn) / 2.f;
63:   const int offsetY = (imageHeight - cropHeightBeforeRtn) / 2.f;
64: 
65:   const uint8_t* yData = (uint8_t*)jniEnv->GetDirectBufferAddress(yBuffer);
66:   const uint8_t* uData = (uint8_t*)jniEnv->GetDirectBufferAddress(uBuffer);
67:   const uint8_t* vData = (uint8_t*)jniEnv->GetDirectBufferAddress(vBuffer);
68: 
69:   float scale = cropWidthAfterRtn / tensorWidth;
70:   int uvRowStride = uRowStride;
71:   int cropXMult = 1;
72:   int cropYMult = 1;
73:   int cropXAdd = offsetX;
74:   int cropYAdd = offsetY;
75:   if (rotateCWDegrees == 90) {
76:     cropYMult = -1;
77:     cropYAdd = offsetY + (cropHeightBeforeRtn - 1);
78:   } else if (rotateCWDegrees == 180) {
79:     cropXMult = -1;
80:     cropXAdd = offsetX + (cropWidthBeforeRtn - 1);
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-120
````cpp
 81:     cropYMult = -1;
 82:     cropYAdd = offsetY + (cropHeightBeforeRtn - 1);
 83:   } else if (rotateCWDegrees == 270) {
 84:     cropXMult = -1;
 85:     cropXAdd = offsetX + (cropWidthBeforeRtn - 1);
 86:   }
 87: 
 88:   float normMeanRm255 = 255 * normMeanRGB[0];
 89:   float normMeanGm255 = 255 * normMeanRGB[1];
 90:   float normMeanBm255 = 255 * normMeanRGB[2];
 91:   float normStdRm255 = 255 * normStdRGB[0];
 92:   float normStdGm255 = 255 * normStdRGB[1];
 93:   float normStdBm255 = 255 * normStdRGB[2];
 94: 
 95:   int xBeforeRtn, yBeforeRtn;
 96:   int yi, yIdx, uvIdx, ui, vi, a0, ri, gi, bi;
 97:   int channelSize = tensorWidth * tensorHeight;
 98:   // A bit of code duplication to avoid branching in the cycles
 99:   if (memoryFormatCode == kMemoryFormatContiguous) {
100:     int wr = outOffset;
101:     int wg = wr + channelSize;
102:     int wb = wg + channelSize;
103:     for (int y = 0; y < tensorHeight; y++) {
104:       for (int x = 0; x < tensorWidth; x++) {
105:         xBeforeRtn = cropXAdd + cropXMult * (int)(x * scale);
106:         yBeforeRtn = cropYAdd + cropYMult * (int)(y * scale);
107:         yIdx = yBeforeRtn * yRowStride + xBeforeRtn * yPixelStride;
108:         uvIdx =
109:             (yBeforeRtn >> 1) * uvRowStride + (xBeforeRtn >> 1) * uvPixelStride;
110:         ui = uData[uvIdx];
111:         vi = vData[uvIdx];
112:         yi = yData[yIdx];
113:         yi = (yi - 16) < 0 ? 0 : (yi - 16);
114:         ui -= 128;
115:         vi -= 128;
116:         a0 = 1192 * yi;
117:         ri = (a0 + 1634 * vi) >> 10;
118:         gi = (a0 - 833 * vi - 400 * ui) >> 10;
119:         bi = (a0 + 2066 * ui) >> 10;
120:         ri = ri > 255 ? 255 : ri < 0 ? 0 : ri;
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-160
````cpp
121:         gi = gi > 255 ? 255 : gi < 0 ? 0 : gi;
122:         bi = bi > 255 ? 255 : bi < 0 ? 0 : bi;
123:         outData[wr++] = (ri - normMeanRm255) / normStdRm255;
124:         outData[wg++] = (gi - normMeanGm255) / normStdGm255;
125:         outData[wb++] = (bi - normMeanBm255) / normStdBm255;
126:       }
127:     }
128:   } else if (memoryFormatCode == kMemoryFormatChannelsLast) {
129:     int wc = outOffset;
130:     for (int y = 0; y < tensorHeight; y++) {
131:       for (int x = 0; x < tensorWidth; x++) {
132:         xBeforeRtn = cropXAdd + cropXMult * (int)(x * scale);
133:         yBeforeRtn = cropYAdd + cropYMult * (int)(y * scale);
134:         yIdx = yBeforeRtn * yRowStride + xBeforeRtn * yPixelStride;
135:         uvIdx =
136:             (yBeforeRtn >> 1) * uvRowStride + (xBeforeRtn >> 1) * uvPixelStride;
137:         ui = uData[uvIdx];
138:         vi = vData[uvIdx];
139:         yi = yData[yIdx];
140:         yi = (yi - 16) < 0 ? 0 : (yi - 16);
141:         ui -= 128;
142:         vi -= 128;
143:         a0 = 1192 * yi;
144:         ri = (a0 + 1634 * vi) >> 10;
145:         gi = (a0 - 833 * vi - 400 * ui) >> 10;
146:         bi = (a0 + 2066 * ui) >> 10;
147:         ri = ri > 255 ? 255 : ri < 0 ? 0 : ri;
148:         gi = gi > 255 ? 255 : gi < 0 ? 0 : gi;
149:         bi = bi > 255 ? 255 : bi < 0 ? 0 : bi;
150:         outData[wc++] = (ri - normMeanRm255) / normStdRm255;
151:         outData[wc++] = (gi - normMeanGm255) / normStdGm255;
152:         outData[wc++] = (bi - normMeanBm255) / normStdBm255;
153:       }
154:     }
155:   } else {
156:     jclass Exception = jniEnv->FindClass("java/lang/IllegalArgumentException");
157:     jniEnv->ThrowNew(Exception, "Illegal memory format code");
158:   }
159: }
160: } // namespace pytorch_vision_jni
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 161-187
````cpp
161: 
162: JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void*) {
163:   JNIEnv* env;
164:   if (vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6) != JNI_OK) {
165:     return JNI_ERR;
166:   }
167: 
168:   jclass c =
169:       env->FindClass("org/pytorch/torchvision/TensorImageUtils$NativePeer");
170:   if (c == nullptr) {
171:     return JNI_ERR;
172:   }
173: 
174:   static const JNINativeMethod methods[] = {
175:       {"imageYUV420CenterCropToFloatBuffer",
176:        "(Ljava/nio/ByteBuffer;IILjava/nio/ByteBuffer;Ljava/nio/ByteBuffer;IIIIIII[F[FLjava/nio/Buffer;II)V",
177:        (void*)pytorch_vision_jni::imageYUV420CenterCropToFloatBuffer},
178:   };
179:   int rc = env->RegisterNatives(
180:       c, methods, sizeof(methods) / sizeof(JNINativeMethod));
181: 
182:   if (rc != JNI_OK) {
183:     return rc;
184:   }
185: 
186:   return JNI_VERSION_1_6;
187: }
````
- EN: Implements callable logic such as `JNI_OnLoad`.
- CN: 实现可调用逻辑，例如 `JNI_OnLoad`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Symbol `imageYUV420CenterCropToFloatBuffer` / 符号 `imageYUV420CenterCropToFloatBuffer`
- Symbol `JNI_OnLoad` / 符号 `JNI_OnLoad`

## Dependencies / 依赖关系
- C/C++ includes: `cassert`, `cmath`, `vector`, `jni.h`
- C/C++ 头文件: `cassert`, `cmath`, `vector`, `jni.h`

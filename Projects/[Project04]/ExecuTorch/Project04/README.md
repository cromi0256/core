[참고 글](https://github.com/meta-pytorch/executorch-examples/tree/main/dl3/android/DeepLabV3Demo)

이 프로젝트는 이미지 분할(Image Segmentation) 모델을 스마트폰으로 배포하여 실행하기까지 다룹니다.

앱 개발의 부분은 코딩 에이전트의 도움을 받았으며 갤럭시 S25 환경(CPU)으로 실행하였습니다.

# 엣지 AI용 모델 생성
```python
import torch
import torchvision.models as models
from executorch.backends.xnnpack.partition.xnnpack_partitioner import XnnpackPartitioner  # (1) CPU 백엔드
from executorch.exir import to_edge_transform_and_lower

model = models.segmentation.deeplabv3_resnet101(weights="DEFAULT").eval()  # (2) 모델 로드
sample_inputs = (torch.randn(1, 3, 224, 224),)

et_program = to_edge_transform_and_lower(  # (3) 모델 내보내기
    torch.export.export(model, sample_inputs),
    partitioner=[XnnpackPartitioner()],
).to_executorch()

with open("dl3_xnnpack_fp32.pte", "wb") as file:
    et_program.write_to_file(file)
```
(1) 백엔드 설정

[갤럭시 S25](https://www.samsung.com/sec/smartphones/galaxy-s25/#performance) 프로세스는 CPU, GPU, NPU를 지원하며 가장 범용적인 CPU 가속기 백엔드로 설정했습니다

(2) 모델 정보

[DeepLabV3](https://pytorch.org/hub/pytorch_vision_deeplabv3_resnet101/) 이미지 분할 모델은 선택하였고, 더 나은 모델은 [YOLO sem](https://docs.ultralytics.com/ko/tasks/semantic#%EB%AA%A8%EB%8D%B8)에서도 확인할 수 있습니다

별도의 양자화를 거치지는 않고 fp32 그대로 모델을 내보냅니다

(3) 모델 내보내기

엣지 디바이스에 최적화 된 모델을 .pte파일로 내려받습니다

# Android 앱 설정
앱을 만들기 위해서 먼저 Android Studio을 다운로드 합니다

새 환경으로부터 시작해 app/assets/ 경로에 만들어둔 모델(dl3_xnnpack_fp.pte)을 넣어둡니다

(1) Android AAR 패키지 설정(디바이스에 executorch 라이브러리 설치)
```kotlin
# app/build.gradle.kts
dependencies {
  implementation("org.pytorch:executorch-android:1.3.1")
}
```
위의 코드로 안드로이드 환경(SDK 34이상)에 executorch 최신버전인 1.3.1을 라이브러리에 추가합니다

```kotlin
repositories {
    google()
    mavenCentral() // 필수 추가
}
```
또한 라이브러리 설치를 위해 mavenCentral도 추가합니다

(2) 메인 설정
```kotlin
package com.example.dl3

import android.content.Context
import android.graphics.Bitmap
import android.graphics.BitmapFactory
import android.graphics.Color
import android.net.Uri
import android.os.Bundle
import android.util.Log
import androidx.activity.ComponentActivity
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.activity.result.PickVisualMediaRequest
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.asImageBitmap
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.unit.dp
import com.example.dl3.ui.theme.Dl3Theme
import org.pytorch.executorch.EValue
import org.pytorch.executorch.Module
import org.pytorch.executorch.Tensor
import java.io.File
import java.io.FileOutputStream
import java.io.InputStream

class MainActivity : ComponentActivity() {
    private var module: Module? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()

        // Load the model from assets
        try {
            val modelPath = assetFilePath(this, "dl3_xnnpack_fp32.pte")
            module = Module.load(modelPath)
            Log.d("ExecuTorch", "Model loaded successfully from $modelPath")
        } catch (e: Exception) {
            Log.e("ExecuTorch", "Error loading model", e)
        }

        setContent {
            Dl3Theme {
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                    DeepLabInferenceScreen(
                        module = module,
                        modifier = Modifier.padding(innerPadding)
                    )
                }
            }
        }
    }

    private fun assetFilePath(context: Context, assetName: String): String {
        val file = File(context.filesDir, assetName)
        if (file.exists() && file.length() > 0) {
            return file.absolutePath
        }
        context.assets.open(assetName).use { inputStream ->
            FileOutputStream(file).use { outputStream ->
                val buffer = ByteArray(4 * 1024)
                var read: Int
                while (inputStream.read(buffer).also { read = it } != -1) {
                    outputStream.write(buffer, 0, read)
                }
                outputStream.flush()
            }
        }
        return file.absolutePath
    }
}

@Composable
fun DeepLabInferenceScreen(module: Module?, modifier: Modifier = Modifier) {
    val context = LocalContext.current
    var selectedBitmap by remember { mutableStateOf<Bitmap?>(null) }
    var resultBitmap by remember { mutableStateOf<Bitmap?>(null) }
    var inferenceResult by remember { mutableStateOf("") }
    
    val pickMedia = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.PickVisualMedia()
    ) { uri ->
        if (uri != null) {
            selectedBitmap = loadBitmapFromUri(context, uri)
            resultBitmap = null
            inferenceResult = "Image selected. Ready for segmentation."
        }
    }

    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(16.dp)
            .verticalScroll(rememberScrollState()),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(text = "DeepLabV3 Image Segmentation", style = MaterialTheme.typography.headlineMedium)
        Spacer(modifier = Modifier.height(16.dp))
        
        Text(text = "Status: ${if (module != null) "Model Loaded" else "Model Load Failed"}")
        
        Spacer(modifier = Modifier.height(16.dp))

        Button(onClick = {
            pickMedia.launch(PickVisualMediaRequest(ActivityResultContracts.PickVisualMedia.ImageOnly))
        }) {
            Text("Pick Image from Gallery")
        }

        Spacer(modifier = Modifier.height(16.dp))

        selectedBitmap?.let { bitmap ->
            Text("Input Image")
            Image(
                bitmap = bitmap.asImageBitmap(),
                contentDescription = "Selected Image",
                modifier = Modifier.size(224.dp).padding(8.dp),
                contentScale = ContentScale.Fit
            )
            
            Spacer(modifier = Modifier.height(16.dp))

            Button(
                onClick = {
                    if (module != null && selectedBitmap != null) {
                        try {
                            val startTime = System.currentTimeMillis()
                            val width = 224
                            val height = 224
                            
                            // 1. Preprocess
                            val inputShape = longArrayOf(1, 3, width.toLong(), height.toLong())
                            val inputData = preprocessBitmap(selectedBitmap!!, width, height)
                            
                            // 2. Inference
                            val inputTensor = Tensor.fromBlob(inputData, inputShape)
                            val inputEValue = EValue.from(inputTensor)
                            val outputs = module.forward(inputEValue)
                            val endTime = System.currentTimeMillis()
                            
                            // 3. Postprocess (DeepLabV3)
                            if (outputs.isNotEmpty()) {
                                val outputTensor = outputs[0].toTensor()
                                val outputData = outputTensor.dataAsFloatArray
                                val shape = outputTensor.shape() // Expected: [1, 21, 224, 224]
                                
                                val numClasses = shape[1].toInt()
                                val outH = shape[2].toInt()
                                val outW = shape[3].toInt()
                                
                                val segmentationBitmap = applyArgmaxAndColor(outputData, numClasses, outH, outW)
                                resultBitmap = segmentationBitmap
                                inferenceResult = "Inference took ${endTime - startTime}ms"
                            }
                        } catch (e: Exception) {
                            inferenceResult = "Error: ${e.message}"
                            Log.e("ExecuTorch", "Inference error", e)
                        }
                    }
                },
                enabled = module != null
            ) {
                Text("Run Segmentation")
            }
        }

        resultBitmap?.let { bitmap ->
            Spacer(modifier = Modifier.height(24.dp))
            Text("Segmentation Result")
            Image(
                bitmap = bitmap.asImageBitmap(),
                contentDescription = "Result Image",
                modifier = Modifier.size(224.dp).padding(8.dp),
                contentScale = ContentScale.Fit
            )
        }

        Spacer(modifier = Modifier.height(24.dp))
        Text(text = inferenceResult, style = MaterialTheme.typography.bodyMedium)
    }
}

fun loadBitmapFromUri(context: Context, uri: Uri): Bitmap? {
    return try {
        val inputStream: InputStream? = context.contentResolver.openInputStream(uri)
        BitmapFactory.decodeStream(inputStream)
    } catch (e: Exception) {
        null
    }
}

fun preprocessBitmap(bitmap: Bitmap, width: Int, height: Int): FloatArray {
    val resizedBitmap = Bitmap.createScaledBitmap(bitmap, width, height, true)
    val floatArray = FloatArray(3 * width * height)
    val mean = floatArrayOf(0.485f, 0.456f, 0.406f)
    val std = floatArrayOf(0.229f, 0.224f, 0.225f)

    for (y in 0 until height) {
        for (x in 0 until width) {
            val pixel = resizedBitmap.getPixel(x, y)
            floatArray[0 * width * height + y * width + x] = ((pixel shr 16 and 0xFF) / 255.0f - mean[0]) / std[0]
            floatArray[1 * width * height + y * width + x] = ((pixel shr 8 and 0xFF) / 255.0f - mean[1]) / std[1]
            floatArray[2 * width * height + y * width + x] = ((pixel and 0xFF) / 255.0f - mean[2]) / std[2]
        }
    }
    return floatArray
}

fun applyArgmaxAndColor(data: FloatArray, numClasses: Int, height: Int, width: Int): Bitmap {
    val bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888)
    val pixels = IntArray(width * height)
    
    // Simple color palette for PASCAL VOC (21 classes)
    val colors = intArrayOf(
        Color.BLACK, Color.RED, Color.GREEN, Color.YELLOW, Color.BLUE, Color.MAGENTA, Color.CYAN,
        Color.GRAY, Color.DKGRAY, Color.LTGRAY, Color.WHITE, Color.rgb(128, 0, 0), Color.rgb(0, 128, 0),
        Color.rgb(128, 128, 0), Color.rgb(0, 0, 128), Color.rgb(128, 0, 128), Color.rgb(0, 128, 128),
        Color.rgb(128, 128, 128), Color.rgb(64, 0, 0), Color.rgb(192, 0, 0), Color.rgb(64, 128, 0)
    )

    for (y in 0 until height) {
        for (x in 0 until width) {
            var maxVal = -Float.MAX_VALUE
            var maxClass = 0
            for (c in 0 until numClasses) {
                // NCHW indexing
                val valIn = data[c * height * width + y * width + x]
                if (valIn > maxVal) {
                    maxVal = valIn
                    maxClass = c
                }
            }
            pixels[y * width + x] = if (maxClass < colors.size) colors[maxClass] else Color.BLACK
        }
    }
    bitmap.setPixels(pixels, 0, width, 0, 0, width, height)
    return bitmap
}
```
프론트엔드는 코딩 에이전트의 도움을 받아 작성하였습니다

코드 작성 후 Sync를 하여야 정상적으로 파일을 읽어올 수 있습니다

MainActivity.kt까지 작성하고 이제 실제 앱으로 배포하는 단계로 넘어갑니다

(2) 개발환경과 디바이스 연동

안드로이드 스튜디오 상단의 디바이스를 Wi-Fi로 연결합니다

Pair Devices Using Wi-Fi를 이용하였으며 USB로도 가능합니다

스마트폰 설정으로 개발자 도구에 접근하여 연결합니다 > QR코드로 연결

성공적으로 연결되었다면 Run 'app' (Shift + F10)을 눌러 스마트폰으로 배포되기까지 잠시 기다립니다(약 1분 소요)

(3) 시연 영상

<img width="220" height="476" alt="Adobe Express - Screen_Recording_20260705_170827_dl3" src="https://github.com/user-attachments/assets/5e03e1c9-ffb8-4ec0-89ef-1bfe74eafdf7" />

ExecuTorch를 사용하여 딥러닝 모델을 성공적으로 스마트폰에 배포하였습니다

엣지 컴퓨팅은 지연시간이 짧고, 네트워크 트래픽 및 대역폭 절감, 보안 및 프라이버시를 향상 할 수 있어 인터넷이 제한된 상황에 효과적입니다

아이폰이나 임베디드 기기에도 적용할 수 있으며

백엔드로 CPU는 물론 GPU와 NPU로도 실핼할 수 있고

LLM모델을 양자화 하여 비용을 절감할 수도 있습니다

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

(2) 개발환경과 디바이스 연

[공식 페이지](https://executorch.ai)

[공식 문서](https://docs.pytorch.org/executorch/main)

ExecuTorch는 모바일, 엣지 디바이스에 파이토치 모델을 효율적으로 실행하기 위한 엔드투엔드 솔루션이다.

# 작동방식
![image](https://docs.pytorch.org/executorch/main/_images/how-executorch-works-high-level.png)
1. 모델 내보내기

   첫 번째 단계는 PyTorch 프로그램을 그래프로 변환한다.
   
   이 그래프는 덧셈, 곱셈, 컨볼루션과 같은 일련의 연산자로 표현할 수 있는 형태다.

   이 과정에서 원본 PyTorch 프로그램 형식을 안전하게 보존한다.

   이러한 변환 방식은 메모리, 연산 능력이 부족한 엣지 환경에서 모델을 실행할 수 있게한다.
   

2. 내보낸 모델을 ExecuTorch 프로그램으로 컴파일

   내보낸 모델을 ‘ExecuTorch 프로그램’으로 실행 가능한 형식으로 변환한다.

   이 단계는 모델의 크기를 줄이기 위한 압축(양자화)부터, 지연 시간을 개선하기 위해 서브그래프를 기기 내 전용 하드웨어 가속기로 추가 컴파일하는 것까지 다양한 최적화를 위한 진입점을 제공한다.

   또한 런타임 메모리 사용량을 줄이기 위해 중간 텐서의 위치를 효율적으로 배치하는 메모리 계획이라 진입점도 제공한다.
   

3. 대상 장치에서 ExecuTorch 프로그램 실행

   입력값(텐서화된 이미)이 주어지면, ExecuTorch 런타임은 ExecuTorch 프로그램을 불러오고, 프로그램이 나타내는 명령어를 실행하여 출력값을 계산한다.

   이 단계가 효율적인 이유는 (1) 런타임이 가볍고 (2) 1단계와 2단계에서 이미 효율적인 실행 계획이 산출되어 있어 고성능 추론이 가능하다.

   또한, 핵심 런타임의 이식성 덕분에 제약이 심한 기기에서도 고성능 실행이 가능하다.

# 코드 예시
[Colab](https://colab.research.google.com/drive/1qpxrXC3YdJQzly3mRg-4ayYiOjC6rue3?usp=sharing#scrollTo=LElQ2xGUkKke)

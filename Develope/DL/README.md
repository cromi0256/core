## 📖 사전지식
* [ML의 전반적인 지식](https://github.com/cromi0256/core/tree/main/ML)
* 파이토치 : [공식문서](https://docs.pytorch.org/tutorials/beginner/basics/intro.html) 가장 활성화 된 딥러닝 프레임워크

## 인공신경망의 활용분야

0. 인공신경망 기초
1. CNN 이미지 처리
2. RNN 자연어 처리
3. 트랜스포머
4. 오토인코더
5. GNN
6. LLM
7. AI Agent
8. 강화학습

## 파이토치 Code
텐서플로우나 JAX도 존재하지만 가장 널리 사용하는 [파이토치](https://docs.pytorch.org/tutorials/beginner/basics/quickstart_tutorial.html)를 선택했다.

[추가자료](https://www.learnpytorch.io/01_pytorch_workflow/)

1. 데이터 준비 : 데이터를 모델이 학습하기 쉬운 텐서로 변환
```python
import torch
from torch import nn
from torch.utils.data import DataLoader
from torchvision import datasets
from torchvision.transforms import v2
# 파이토치는 이미지처리를 위한 torchvision외에 torchaudio도 존재한다.


# Download training data from open datasets.
training_data = datasets.FashionMNIST(
    root="data",
    train=True,
    download=True,
    transform=v2.Compose([v2.ToImage(), v2.ToDtype(torch.float32, scale=True)]),
)

# Download test data from open datasets.
test_data = datasets.FashionMNIST(
    root="data",
    train=False,
    download=True,
    transform=v2.Compose([v2.ToImage(), v2.ToDtype(torch.float32, scale=True)]),
)


batch_size = 64

# Create data loaders.
train_dataloader = DataLoader(training_data, batch_size=batch_size)
test_dataloader = DataLoader(test_data, batch_size=batch_size)

for X, y in test_dataloader:
    print(f"Shape of X [N, C, H, W]: {X.shape}")
    print(f"Shape of y: {y.shape} {y.dtype}")
    break
```

2. 모델 설계 : 모델 파라미터를 설정
```python
device = torch.accelerator.current_accelerator().type if torch.accelerator.is_available() else "cpu"
print(f"Using {device} device")
# 더 빠른 연산을 위한 장치 설정

# Define model
class NeuralNetwork(nn.Module):
    def __init__(self):
        super().__init__()
        self.flatten = nn.Flatten()
        self.linear_relu_stack = nn.Sequential(
            nn.Linear(28*28, 512),
            nn.ReLU(),
            nn.Linear(512, 512),
            nn.ReLU(),
            nn.Linear(512, 10)
        )

    def forward(self, x):
        x = self.flatten(x)
        logits = self.linear_relu_stack(x)
        return logits

model = NeuralNetwork().to(device)
print(model)
# 가속 연산을 위해 모델을 사용가능한 장치로 이동
```

3. 모델 훈련 : 배치 단위로 모델을 학습
```python
loss_fn = nn.CrossEntropyLoss()
optimizer = torch.optim.SGD(model.parameters(), lr=1e-3)
# 손실함수와 옵티마이저 설정

def train(dataloader, model, loss_fn, optimizer):
    size = len(dataloader.dataset)
    model.train()
    for batch, (X, y) in enumerate(dataloader):
        X, y = X.to(device), y.to(device)

        # Compute prediction error
        pred = model(X)  # 순전파
        loss = loss_fn(pred, y)  # 손실 계산

        # Backpropagation
        loss.backward()  # 역전파
        optimizer.step()  # 파라미터 조정
        optimizer.zero_grad()  # 그래디언트 초기화

        if batch % 100 == 0:
            loss, current = loss.item(), (batch + 1) * len(X)
            print(f"loss: {loss:>7f}  [{current:>5d}/{size:>5d}]")
```

4. 모델 추론 : 이 단계에는 그래디언트 계산을 하지 않음
```python
def test(dataloader, model, loss_fn):
    size = len(dataloader.dataset)
    num_batches = len(dataloader)
    model.eval()
    test_loss, correct = 0, 0
    with torch.no_grad():
        for X, y in dataloader:
            X, y = X.to(device), y.to(device)
            pred = model(X)
            test_loss += loss_fn(pred, y).item()
            correct += (pred.argmax(1) == y).type(torch.float).sum().item()
    test_loss /= num_batches
    correct /= size
    print(f"Test Error: \n Accuracy: {(100*correct):>0.1f}%, Avg loss: {test_loss:>8f} \n")


epochs = 5
for t in range(epochs):
    print(f"Epoch {t+1}\n-------------------------------")
    train(train_dataloader, model, loss_fn, optimizer)
    test(test_dataloader, model, loss_fn)
print("Done!")
```

5. 모델 저장&로드 : 파라미터를 체크포인트로 저장
```python
# 모델 저장
torch.save(model.state_dict(), "model.pth")
print("Saved PyTorch Model State to model.pth")

# 모델 로드
model = NeuralNetwork().to(device)
model.load_state_dict(torch.load("model.pth", weights_only=True))
```
## 관련링크
[파이토치](https://pytorch.org)

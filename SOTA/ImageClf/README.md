[참고자료1](https://pytorch.org/blog/how-to-train-state-of-the-art-models-using-torchvision-latest-primitives/)

[참고자료2](https://docs.pytorch.org/tutorials/beginner/transfer_learning_tutorial.html)

![Image](https://pytorch.org/wp-content/uploads/2024/11/Cumulative20Accuracy20Improvements20for20ResNet50.png)

토치비전 V2 weight는 위의 레시피로 만들어졌다.

학습 방식에 변화를 주어 ResNet50의 정확도를 4.7% 상승한 방법을 알아보자.

## 베이스라인

[코드 참조](https://github.com/pytorch/vision/blob/main/references/classification/train.py)

```Python
  batch_size=4

  epochs=15
  opt='sgd',  
  momentum=0.9

  lr=0.001
  lr_scheduler='steplr'
  lr_step_size=5
  lr_gamma=0.1


  # Regularization
  weight_decay=1e-4


  # Resizing
  interpolation='bilinear'
  val_resize_size=256
  val_crop_size=224
  train_crop_size=224

  # v2.transform
  train_transforms = v2.Compose([
    v2.ToImage(),
    v2.RandomResizedCrop(size=(224,224), interpolation=InterpolationMode.BILINEAR, antialias=True),
    v2.RandomHorizontalFlip(0.5),
    v2.ToDtype(torch.float, scale=True),
    v2.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]),
    v2.ToPureTensor()
  ])
```

## 학습률 최적화

```python
  lr=0.5, 
  lr_scheduler='cosineannealinglr', 
  lr_warmup_epochs=5, 
  lr_warmup_method='linear', 
  lr_warmup_decay=0.01,
```
웜업과 코사인어닐링을 결합한 동적인 학습률을 지정

## Trivial Augment

```python
auto_augment='ta_wide',
```
단순한 변형을 1번 적용

## Long Training

```python
epochs=600,
```
학습 에포크 증가

## 무작위 지움

```python
random_erase=0.1,
```
이미지의 일부를 지워 드롭아웃같은 효과를 냄

## 라벨 스무딩

```python
label_smoothing=0.1,
```
모델이 과도한 확신을 줄여 과적합 방지

## MixUp & CutMix

```python
mixup_alpha=0.2, 
cutmix_alpha=1.0,
```
한 이미지에 다른 이미지를 일부 섞어 학습

## 가중치 감쇠 튜닝

```python
weight_decay=2e-05, 
norm_weight_decay=0.0,
```
전역적 파라미터에 정규화 정도를 조정

## FixRes

```python
val_crop_size=224, 
train_crop_size=176,
```
추론에는 더 높은 해상도로 평가하여 성능향상

## EMA

```python
model_ema=True, 
model_ema_steps=32, 
model_ema_decay=0.99998,
```
지수이동평균으로 가중치를 조절하여 일반화와 안정화에 기여

## 추론 리사이즈 튜닝

```python
val_resize_size=232,
```
훈련에는 256로 리사이즈 후 224로 크롭했다면, [224, 256] 범위의 크기로 리사이즈 하여 이미지 확대 왜곡 효과 절감

## 반복 증강

```python
ra_sampler=True,
ra_reps=4,
```
하나의 이미지를 여러번 샘플링하여 변형시킨 후 학습

## 그외에...

소개한 방법으로는 단일 모델을 사용했지만 더 나은 백본 모델이나 앙상블로 더 향상시킬 수 있다

[허깅페이스](https://huggingface.co/learn/cookbook/fine_tuning_vit_custom_dataset)의 ViT파인튜닝

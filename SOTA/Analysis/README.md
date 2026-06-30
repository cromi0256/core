데이터-모델 분석 시각화 정리

# 데이터 분석
1. 단변량 분석
   A. 카테고리형 변수
     - 구성 : 샘플 수, 카테고리 수
     - 시각화 : 막대그래프
     - 코드 : sns.catplot(data, x, kind="count")
       
   B. 연속형 변수
     - 구성 : 샘플 수, 값, 데이터 분포(평균, 중앙값, 분산...)
     - 시각화 : 히스토그램
     - 코드 : sns.distplot(data, x)

2. 다변량 분석
   A. (cat - cat) 변수
     - 시각화 : 막대그래프
     - 코드 : sns.catplot(data, x=, hue=, kind="count")

   B. (cat - con) 변수
     - 시각화 : swarm 플롯(수 + 값) + 바이올린 플롯(데이터 분포)
     - 코드 :
     g = sns.catplot(data, x, y, kind="violin") 
     sns.swarmplot(data, x, y, ax=g.ax)
     
   C. (con - con) 변수
     - 시각화 : 산점도
     - 코드 : sns.relplot(data, x, y)
  
   D. 시계열 데이터
     - 구성 : 타임스탬프, 값
     - 시각화 : 선 그래프
     - 코드 : sns.relplot(data, x, y, kind="line")
  
   E. 셋이상의 변수
     - 시각화 : 산점도(hue), 선그래프(hue)


# 모델 분석
  - 구성 : 모델 점수, 파라미터, 훈련-추론 시간
  - 시각화 : 혼동 행렬, ROC-AUC, 오차분석
  

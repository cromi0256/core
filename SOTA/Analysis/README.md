데이터-모델 분석 시각화 정리

- 정적 이미지 저장

  plt.savefig("plot.svg", bbox_inches="tight")

- 동적 이미지 저장
  ```Python
  df = px.data.tips()
  fig = px.scatter(df, x="total_bill", y="tip", color="sex")
  fig.write_html("scatter.html")
  ```
- mlflow 아티팩트 저장

  [단일]mlflow.log_figure(fig, "plots/scatter.png")
  
  [폴더]mlflow.log_artifact(filepath, artifact_path="plots")

# 데이터 분석
1. 단변량 분석
   A. 카테고리형 변수
     - 구성 : 샘플 수, 카테고리 수
     - 시각화 : 막대그래프
     - 코드 : sns.catplot(data, x, kind="count")

       ![Image](https://seaborn.pydata.org/_images/categorical_41_0.png)
       
   B. 연속형 변수
     - 구성 : 샘플 수, 포인트(값), 데이터 분포(평균, 중앙값, 분산...)
     - 시각화 : 히스토그램
     - 코드 : sns.distplot(data, x)
  
       ![Image](https://seaborn.pydata.org/_images/distributions_3_0.png)

2. 다변량 분석
   A. (cat - cat) 변수
     - 시각화 : 막대그래프
     - 코드 : sns.catplot(data, x=, hue=, kind="count")
  
       ![image](https://seaborn.pydata.org/_images/categorical_37_0.png)

   B. (cat - con) 변수
     - 시각화 : swarm 플롯(수 + 값) + 바이올린 플롯(데이터 분포)
     - 코드 :
       ```
       g = sns.catplot(data, x, y, kind="violin")
       sns.swarmplot(data, x, y, ax=g.ax)
       ```

       ![image](https://seaborn.pydata.org/_images/categorical_35_0.png)
     
   C. (con - con) 변수
     - 시각화 : 산점도
     - 코드 : sns.relplot(data, x, y)
  
       ![image](https://seaborn.pydata.org/_images/relational_4_0.png)
  
   D. 시계열 데이터
     - 구성 : 타임스탬프, 값
     - 시각화 : 선 그래프
     - 코드 : sns.relplot(data, x, y, kind="line")
  
       ![image](https://seaborn.pydata.org/_images/relational_21_0.png)
  
   E. 셋이상의 변수
     - 시각화 : 산점도(hue), 선그래프(hue)
  
       ![image](https://seaborn.pydata.org/_images/relational_55_0.png)


# 모델 분석
  - 구성 : 모델 점수, 파이프라인, 파라미터, 훈련-추론 시간
  - 시각화 : 혼동 행렬, ROC-AUC, 오차분석...
  - 코드 : [출처](https://scikit-learn.org/stable/model_selection.html)
  
    metrics.ConfusionMatrixDisplay(...[, ...])
    
    metrics.RocCurveDisplay(*, fpr, tpr[, ...])
    
    [사이킷런 1.9.0]metric_at_thresholds

    
# 모델-데이터 분석
   - 구성 : 피쳐중요도, 결과해석, 분산-편향 검토
   1. Permutation Feature Importance
      
   3. SHAP

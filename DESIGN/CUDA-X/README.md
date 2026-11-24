# [CUDA-X](https://developer.nvidia.com/topics/ai/data-science/cuda-x-for-data-science?size=n_6_n&sort-field=featured&sort-direction=desc)

![image](https://www.nvidia.com/content/dam/en-zz/Solutions/deep-learning/software/Rapids/nvidia-rapids-data-processing-evolution-diagram.svg)

CUDA-X™는 CUDA™를 기반으로 구축된 고도로 최적화된 도메인 특화 라이브러리 모음으로, 데이터 과학 가속화를 위한 오픈 소스 라이브러리 제품군을 포함합니다.

데이터 과학 생태계 내 100개 이상의 오픈 소스 라이브러리 및 도구와의 연동 기능과, pandas 및 scikit-learn과 같은 널리 사용되는 PyData 도구의 성능을 가속화하는 코드 변경이 필요 없는 API를 통해, 데이터 과학자들은 기존 도구를 활용하여 워크플로우를 손쉽게 가속화할 수 있습니다.

## CUDA-X 라이브러리
[공식 문서](https://docs.nvidia.com/datascience/)
### cuDF
[공식 문서](https://docs.nvidia.com/cudf/latest/)

판다스, 폴라스, 아파치 스파크 같은 데이터 프레임 처리를 GPU로 가속한다.

[코랩](https://colab.research.google.com/github/rapidsai-community/showcase/blob/main/accelerated_data_processing_examples/polars_gpu_engine_demo.ipynb?ncid=so-othe-465994-vt27)

### cuML
[공식 문서](https://docs.nvidia.com/cuml/latest/)

기존에 CPU로 작업하던 머신러닝을 GPU로 가속한다.

모든 사이킷런 모델이 구현되어 있지 않으며 자세한 내용은 API문서를 찾아본다.

[성능 벤침마크](https://docs.nvidia.com/cuml/latest/cuml-accel/benchmarks/)

### cuGraph
[공식문서](https://docs.nvidia.com/cugraph/latest/)

그래프 분석을 위한 그래프 알고리즘 최적화 솔루션.

[코랩](https://colab.research.google.com/github/rapidsai-community/showcase/blob/main/getting_started_tutorials/accelerated_networkx_demo.ipynb?ncid=so-othe-145635-vt27)

## 이외에도...
[문서](https://docs.nvidia.com/datascience/api/)
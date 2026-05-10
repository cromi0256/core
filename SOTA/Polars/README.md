## 소개
[폴라스](https://pola.rs/)

폴라스는 판다스와 같이 데이터 조작에 쓰이며 효율적인 병렬처리로 다른 툴보다 월등히 빠르다.

판다스 명령어와 비슷하면서도 배우기 쉬워 차세대 표준이 될 것이다.

```PowerShell
pip install polars
```

```py
import polars as pl

df = pl.read_csv('./train.csv')

print(df.head())
print(df.tail())

print(df.schema)  # pandas .info 와 유사
print(df.describe())
```

## 기본 명령어
자세한 정보는 [공식문서](https://docs.pola.rs/)를 확인

`select` : 특정 열의 데이터를 선택하여 조작할때 쓰임

`with_columns` : 기존 데이터프레임에 추가하여 조작할때 쓰임

`filter` : 특정 조건에 맞는 데이터를 필터링

`group_by` : 그룹화 조건에 따라 여러 행들을 병합

`join` : 두 그룹을 on과 how에 따라 조인

`concat` : 여러 데이터프레임을 연결

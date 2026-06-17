# 데이터분석 — Pandas 기초 정리

## 1. Pandas란?

파이썬의 대표적인 데이터 분석 라이브러리로, 데이터 조작·분석·시각화를 손쉽게 처리할 수 있다.

---

## 2. 핵심 데이터 구조

### 2.1 Series (1차원)

하나의 열(column)과 같은 1차원 배열 구조.

```python
import pandas as pd

s = pd.Series([10, 20, 30, 40])
print(s)
# 0    10
# 1    20
# 2    30
# 3    40
```

### 2.2 DataFrame (2차원)

여러 열을 가진 테이블 구조. (실제 데이터 분석에서 가장 많이 사용한다.)

```python
import pandas as pd

df = pd.DataFrame({
    "name": ["Kim", "Lee", "Park"],
    "age": [20, 22, 21],
    "score": [85, 90, 78]
})
print(df)
#    name  age  score
# 0   Kim   20     85
# 1   Lee   22     90
# 2  Park   21     78
```

---

## 3. CSV 파일 다루기
- **csv(Comma Separated Values) 파일**: 데이터를 쉼표로 구분해 저장한 텍스트 파일. 별도의 데이터 타입이 없고 모든 값이 문자열로 취급된다.

### 3.1 불러오기

```python
df = pd.read_csv("파일명.csv")
```

### 3.2 행 출력

| 함수 | 설명 |
|------|------|
| `df.head(n)` | 상위 n개 행 출력 (기본값: 5) |
| `df.tail(n)` | 하위 n개 행 출력 (기본값: 5) |
| `df[ df['컬럼명'] > 값 ]` | 특정 조건에 맞는 행 선택 (행 필터링) |
| `df[ (df['컬럼명'] > 값 ) & (df['컬럼명'] == '값']` | 복합 조건 |
| `df.drop([행 인덱스], axis=0)` | 특정 행 삭제 후 반환 |

**복합 조건 연산자**
- `&`: AND
- `|`: OR
- `~`: NOT

### 3.3 열 선택

| 표현식 | 반환 타입 | 설명 |
|--------|-----------|------|
| `df.columns` | Index | 전체 컬럼명 확인 |
| `df['컬럼명']` | Series | 단일 컬럼 선택 |
| `df[['컬럼명']]` | DataFrame | 단일 컬럼을 DataFrame으로 선택 |
| `df[['컬럼1', '컬럼2']]` | DataFrame | 여러 컬럼 선택 |
| `df.drop(['컬럼명'], axis=1)` | DataFrame | 특정 컬럼 삭제 후 반환 |

### 3.4 크기 확인

| 함수 | 설명 |
|------|------|
| `df.shape` | (행 수, 열 수) 튜플 반환 |
| `df.size` | 전체 원소 개수 반환 |

### 3.5 데이터 정보 확인            

| 함수 | 설명 |
|------|------|
| `df.info()` | 컬럼별 타입·결측치 등 전반적인 정보 출력 |
| `df.info(memory_usage='deep')` | 메모리 사용량 더 정확하게 출력 |
| `df.describe()` | 수치형 컬럼의 요약 통계(평균·표준편차 등) 출력 |

### 3.6 결측치 확인

```python
df.isnull()             # 결측치 위치를 True/False로 반환
df.isnull().sum()       # 열(컬럼)별 결측치 개수
df.isnull().sum(axis=1) # 행별 결측치 개수
df.isnull().sum().sum() # 전체 결측치 총 개수
```

### 3.7 결측치 처리

| 함수 | 설명 |
|------|------|
| `df.fillna(값)` | 결측치를 특정 값으로 채우고 새로운 DataFrame 반환 |
| `df.fillna(값, inplace=True)` | 원본 DataFrame의 결측치를 특정 값으로 채우기 |

### 3.8 정렬

| 함수 | 설명 |
|------|------|
| `df.sort_values("컬럼명")` | 오름차순 |
| `df.sort_values("컬럼명", ascending=False)` | 내림차순 |
| `df.sort_values(["컬럼명1", "컬럼명2"] )` | 컬럼1 기준 오름차순 → 컬럼2 기준 오름차순 |
| `df.sort_values(["컬럼명1", "컬럼명2"], ascending=[False, True] )` | 컬럼1 기준 내림차순 → 컬럼2 기준 오름차순 |

---

## 4. 데이터 변환

| 함수 | 설명 |
|------|------|
| `s.to_frame()` | Series → DataFrame 변환 |
| `df.to_csv("파일명.csv")` | DataFrame → CSV 저장 |
| `df.to_csv("파일명.csv", index=False)` | 인덱스 제외하고 CSV 저장 |

---

## 5. Pandas 기초 조작: 컬럼 선택, 행 필터링, 정렬, 조건문, 새 컬럼 만들기
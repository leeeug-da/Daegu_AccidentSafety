# 대구 교통사고 예측모델 개발 

### ✨기획의도
이동수단의 발달에 따라 다양한 유형의 교통사고들이 계속 발생하고 있음. 해당 사고의 원인을 규명하고 사고율을 낮추기 위해, 다양한 시공간 정보를 융합하여 사고위험도(ECLO)를 예측해 보고자 함

`ECLO (Equivalent Casualty Loss Only)`

- 인명피해 심각도를 기반한 사고위험 지표
- 계산식 : 
  - 사망자수 X 10 + 중상자수 X 5 + 경상자수 X 3 + 부상자수 X 1

<br>

### ⚙️ 개발도구

- `Tool` : `Pandas` `Numpy` `Seaborn` `Matplotlib` `Sklearn` `Scipy`
- `Language` : `Python`

<br>

### 📂 DATA
데이콘 경진대회에서 제공한 데이터 활용

<br>

### 🛣️ 분석방향

- EDA 및 현황 분석을 통해 사고위험도와 관련 있는 위험 요인(컬럼) 찾기
- 사고위험도 (ECLO) 예측 모델 개발 및 평가 (RMSLE score)

<br>

### 💡 진행과정

**1. 데이터 전처리**

- 위험 요인을 더 면밀하게 분석 하기 위해 파생변수 생성
- 범주형 데이터 원핫 인코딩 진행 

<br>

**2. 사고위험도와 관련 있는 요인은 무엇인가?** 

- 행정 구역 정보와 다른 피처들간의 상관관계를 파악한 결과, 유사한 그래프 패턴이 나타남을 파악. 이는 **“행정 구역” 정보 자체에 사고위험도와 관련 있는 두드러지는 특징이 있음**을 시사.
    
    
    ![현황 분석 1) `행정 구역 ↔ 도로형태 ↔ ECLO` 상관관계 ](https://github.com/leeeug-da/Daegu_AccidentSafety/blob/main/DATA/images/%EA%B3%B5%EA%B0%84%EC%A0%95%EB%B3%B4%EC%8B%9C%EA%B0%81%ED%99%94_%EB%8F%84%EB%A1%9C.png)
    
    현황 분석 1) `행정 구역 ↔ 도로형태 ↔ ECLO` 상관관계 
    
    ![현황 분석 2) `행정 구역 ↔ 피해자 연령대 ↔ ECLO` 상관관계](https://github.com/leeeug-da/Daegu_AccidentSafety/blob/main/DATA/images/%EA%B3%B5%EA%B0%84%EC%A0%95%EB%B3%B4%EC%8B%9C%EA%B0%81%ED%99%94.png)
    
    현황 분석 2) `행정 구역 ↔ 피해자 연령대 ↔ ECLO` 상관관계

<br>

- 유사한 수치를 가진 구역들을 그룹화하여, 리서치를 통해 사고위험도와 관련된 특성의 데이터를 추가로 확보한 결과, 다음과 같은 결론을 도출
    
    
    **1그룹**
    *📊 사고 발생률이 낮지만 평균 ECLO가 높음*
    - 고령층이 주로 거주
    - 산업단지
    → **높은 사고 위험도**
    
    **2그룹**
    *📊 사고 발생률이 높지만 평균 ECLO가 낮음*
    - 거주인구가 많은 지역
    - 반면 교통 관련 통제가 잘 되어 있을 확률이 높음
    → *사고에 대한 예방이 잘 이루어지고 있다는 것을 나타냄*
    → **비교적 낮은 사고 위험도**
    
    **3그룹**
    *📊 사고 발생률과 평균 ECLO가 모두 낮음*
    - 번화가가 위치해 있는 지역
    - 대중교통의 활발한 이용
    - 사고 위험도가 낮은 젊은층이 주 유동인구
    → *위험도가 높은 사고를 유발하지 않음을 의미*
    → **낮은 사고 위험도**


 <br>
 
- 이러한 현황 분석 결과를 바탕으로, 각 그룹의 특성을 고려하여 모델의 예측력을 향상시키기 위해 **행정 구역에 대한 상대적인 가중치**를 부여
    
    
    <ins>**가중치 계산식**</ins>
    
    $$
    eclo 평균 / 최대 eclo 평균
    $$
    
    → 목적 : 모델이 중요한 특징에 대해 더 집중할 수 있도록  
    
    → 최대 ECLO 평균은 데이터에서 가장 큰 ECLO 값을 나타냄
    
    → 이 값으로 ECLO의 범위를 정의
    
    <ins>**결과**</ins>
    
    ![ECLO.png](https://github.com/leeeug-da/Daegu_AccidentSafety/blob/main/DATA/images/ECLO.png)
    

<br>

**3. 1차 모델링 후, 피처 엔지니어링을 진행하여 모델 선능 개선** 

![스크린샷 2024-04-18 오후 4.06.37.png](https://github.com/leeeug-da/Daegu_AccidentSafety/blob/main/DATA/images/ECLO_Log.png)

- 각 함수에 대해 로그변환 후 ECLO 값에 대한 왜도가 있음을 확인 → 왜도값이 가장 낮은 log2 변환 방식 채택

<br>

![스크린샷 2024-04-18 오후 4.04.45.png](https://github.com/leeeug-da/Daegu_AccidentSafety/blob/main/DATA/images/feature_engineering.png)

- Randomized Search 를 통한 최적의 parameters 찾기

<br>                                                                               

**4. 모델링 결과 (RMSLE score) 상위 2개 모델 선정**

![스크린샷 2024-04-18 오후 3.46.21.png](https://github.com/leeeug-da/Daegu_AccidentSafety/blob/main/DATA/images/RMSLE_score.png)

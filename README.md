# 서울시 노인 돌봄 서비스 수요 예측 및 사각지대 분석
![image](기사 사진)
## 프로젝트 개요
### 프로젝트 배경
<pre>
고령화 사회로의 진입에 따라 노인 복지 수요가 증가하고 있으며, 지역 간 복지 서비스 격차도 심화되고 있음.
서울시는 데이터 접근성 높고 지역 간 복지 격차가 뚜렷하므로 서울시 데이터를 기반으로 분석을 진행함.
</pre>
### 프로젝트 목표
<pre>
서울시 자치구 단위 데이터를 기반으로 노인 돌봄 서비스 수요를 예측하고, 복지 사각지대를 군집 분석을 통해 탐색하고 정책적 시사점을 도출
<기사출처>
https://www.ebn.co.kr/news/articleView.html?idxno=1647664
</pre>

---
## 1. 폴더 및 파일 설명
### 폴더 설명
<pre>
data/ : 분석에 활용된 공공데이터(.csv, .xlsx)를 저장한 폴더
    </pre>
### 파일 설명
#### EDA.ipynb
<pre>
EDA 분석 수행
- 주요 작업
    - 각 변수의 분포 시각화
    - 결측치 및 이상치 확인
    - 지역 간 변수 비교
    </pre>
#### data_preprocessing.ipynb
<pre>
다수의 원본 데이터를 통합하고 정제하는 전처리 과정 수행
- 주요 작업
    - 법정동/행정동 이름 정규화
    - 필요한 열만 선택하여 새로운 데이터프레임 생성
    - 결측치 제거, 정렬
    - Z-score 표준화
    </pre>
#### clustering.ipynb
<pre>
클러스터링 분석 수행
- 사용 기법 : KMeans
- 주요 작업
    - 자치구별 인프라·복지 지표를 기반으로 유사한 지역 군집화
    - 최적 클러스터 수 결정 (예: Elbow Method, Silhouette Score)
    - 분석 결과를 바탕으로 정책 제안 및 인사이트 도출
        - 클러스터링 결과 바탕으로 대표 자치구 선정
        - 복지 인프라 및 취약성 지표에 기반한 정책 도입 우선순위 도출
    </pre>
#### sub_clustering.ipynb
<pre>
cluster1에 포함된 지역만 따로 재분석하여 세분화
- 목적 : 세부 지역 간 미묘한 차이를 파악하고 맞춤형 정책 인사이트 도출
- 주요 작업
    - cluster 1 안에서 3개의 하위 클러스터로 분류
    - 지도 시각화와 변수 분석을 통해 각 집단 특성 비교
</pre>

</pre>

## 2. 활용 Data 소개
### 데이터 출처
- [서울 열린데이터 광장](https://data.seoul.go.kr)
  - 노인인구, 복지관 현황, 의료복지시설 수, 정류장/노선 수, 요양보호사 수 등
### 데이터 단위
- 행정동 단위로 데이터 수집 및 통일
### 사용한 데이터
<pre>
- 자살률(구별).xlsx : 자치구별 전체 자살률 기반, 노인 자살률 추정
- 독거노인 인구.xlsx : 자치구별 독거노인 및 기초 수급자 수
- 노인의료복지시설 현황.xlsx : 자치구별 노인을 위한 의료 복지시설 개수
- 재가노인복지시설 현황.xlsx : 자치구별 재가 복지 서비스 제공 시설수
- 노인여가복지시설 현황.xlsx : 노인 여가시설 현황 및 종사자 수 등
- 요양보호사 성별 현황.csv : 자치구별 등록된 요양보호사 수
- 정류소현황(2019~2023).xlsx : 교통 접근성: 자치구별 정류소 수, 평균 노선 수
- 서울시 노인 교통사고 통계.xlsx : 노인 대상 보행 사고 통계 수치
    </pre>
![image](사진 자료 : 각 엑셀 파일 캡처본)
### 사용한 변수 목록
<pre>
- 1인당 복지관 수
- 1인당 노인의료복지시설 수
- 1인당 재가노인복지시설 수
- 요양보호사 수
- 1인당 복지관종사자 수
- 1인당 정류장 수
- 평균 노선 수
- 1인당 독거노인 수
- 추정 노인 자살률
    </pre>
![image](사진 및 코드 첨부)

## 3. 데이터 전처리
- 변수별 처리 내용
    (1) 고령자 인구 & 노인 자살률 : 전체 자살률 × 전국 노인 비중으로 추정치 계산
    ```python
    df_raw =pd.read_excel("/content/drive/MyDrive/코랩/데이터마이닝/자살률(구별).xlsx")

    # 전국 평균 자살률 정보 (단위: 명 / 10만명)
    national_elderly_suicide_rate = 40.6
    national_total_suicide_rate = 27.3

    # 자치구별 자살률 데이터 불러오기 및 0행 제거
    df = df_raw.iloc[1:].copy()  # 0행(소계) 제거

    # 필요한 컬럼 추출 및 이름 변경
    df = df[['자치구별', '자살률 (10만명당 명)']]
    df.rename(columns={
        '자치구별': '자치구',
        '자살률 (10만명당 명)': '전체 자살률'
    }, inplace=True)

    # 3️자치구별 추정 노인 자살률 열 추가
    df['추정 노인 자살률'] = df['전체 자살률'] * (national_elderly_suicide_rate / national_total_suicide_rate)

    df = df[['자치구', '추정 노인 자살률']]

    df
    ```
    (2) 독거노인, 저소득노인, 기초수급자 수: 수치형 변환 및 병합
    ```python
    lonely_df = pd.read_excel("/content/drive/MyDrive/코랩/데이터마이닝/팀플/data/독거 노인 인구.xlsx")

    lonely_cleaned = lonely_df.iloc[1:].copy()

    # 필요한 열 선택 및 열 이름 변경
    lonely_cleaned = lonely_cleaned[['자치구', '소계', '국민기초생활보장 수급권자', '저소득노인']]
    lonely_cleaned.columns = ['자치구', '독거노인 합계', '기초수급자 합계', '저소득노인 합계']

    # 수치형으로 변환
    lonely_cleaned[['독거노인 합계', '기초수급자 합계',     '저소득노인 합계']] = lonely_cleaned[
    ['독거노인 합계', '기초수급자 합계', '저소득노인 합계']
    ].apply(pd.to_numeric, errors='coerce')

    # df에 중복 열이 있을 경우 삭제
    for col in ['독거노인 합계', '기초수급자 합계', '저소득노인 합계']:
        if col in df.columns:
            df = df.drop(columns=[col])

    # 자치구 기준으로 병합
    df = pd.merge(df, lonely_cleaned, on='자치구', how='left')

    ```
    (3) 복지시설(의료, 재가, 여가): 자치구별 시설 수 count 후 병합
    ```python
    medical_df = pd.read_excel("/content/drive/MyDrive/코랩/데이터마이닝/팀플/data/서울시 노인의료복지시설현황.xlsx")

    facilities = medical_df.iloc[3:].copy()

    # 자치구 열 이름 지정 및 정리
    facilities = facilities.rename(columns={'Unnamed: 1': '자치구'})
    facilities['자치구'] = facilities['자치구'].astype(str).str.strip()

    facilities = facilities[facilities['자치구'].str.contains("구", na=False)]

    # 자치구별 시설 수 계산
    facility_counts = facilities['자치구'].value_counts().reset_index()
    facility_counts.columns = ['자치구', '노인의료복지시설 수']

    df['자치구'] = df['자치구'].astype(str).str.strip()

    # 병합 전 동일 열이 있으면 제거
    if '노인의료복지시설 수' in df.columns:
        df = df.drop(columns=['노인의료복지시설 수'])

    # 병합
    df = pd.merge(df, facility_counts, on='자치구', how='left')

    # 9. NaN → 0 처리
    df['노인의료복지시설 수'] = df['노인의료복지시설 수'].fillna(0).astype(int)

    df
    ```
    ```python
    homecare_df = pd.read_excel("/content/drive/MyDrive/코랩/데이터마이닝/팀플/data/서울시 재가노인 복지시설 현황.xlsx")

    # 자치구와 시설 수(합계) 열만 추출
    homecare_facilities = homecare_df[['자치구', '합계']].copy()

    homecare_facilities['자치구'] = homecare_facilities['자치구'].astype(str).str.strip()

    homecare_facilities['합계'] = pd.to_numeric(homecare_facilities['합계'], errors='coerce')

    # 열 이름 변경
    homecare_facilities = homecare_facilities.rename(columns={'합계': '재가노인 복지시설 수(개소)'})

    df['자치구'] = df['자치구'].astype(str).str.strip()

    # 병합 전 동일 열이 있으면 제거
    if '재가노인 복지시설 수(개소)' in df.columns:
        df = df.drop(columns=['재가노인 복지시설 수(개소)'])

    df = pd.merge(df, homecare_facilities, on='자치구', how='left')

    # NaN 값은 0으로 처리
    df['재가노인 복지시설 수(개소)'] = df['재가노인 복지시설 수(개소)'].fillna(0).astype(int)

    # 불필요한 열 제거
    if '재가노인 복지시설 수' in df.columns:
        df = df.drop(columns=['재가노인 복지시설 수'])

    # 열 이름 변경
    df = df.rename(columns={'재가노인 복지시설 수(개소)': '재가노인복지시설 수'})
    df
    ```
    ```python
    leisure_df = pd.read_excel("/content/drive/MyDrive/코랩/데이터마이닝/팀플/data/서울시 노인여가 복지시설 현황 .xlsx")

    # 필요한 열만 추출하고 이름 정리
    leisure_df = leisure_df[['자치구', '노인복지관 수', '복지관종사자 수', '경로당 수', '노인교실 수']]
    leisure_df.rename(columns={
    '노인복지관 수': '복지관 수',
    '복지관종사자 수': '복지관종사자 수',
    '경로당 수': '경로당 수',
    '노인교실 수': '노인교실 수'
    }, inplace=True)

    # df에 네 가지 열을 자치구 기준으로 붙이기
    for col in ['복지관 수', '복지관종사자 수', '경로당 수', '노인교실 수']:
        df[col] = df['자치구'].map(leisure_df.set_index('자치구')[col])

    df
    ```
    (4) 요양보호사 수: 결측치(중랑구)는 전체 평균으로 대체
    ```python
    caregiver_df = pd.read_csv("/content/drive/MyDrive/코랩/데이터마이닝/팀플/data/서울시 요양보호사 남녀별 자격 현황정보.csv", encoding='cp949')

    gu_names = caregiver_df['자치구명'].unique()
    print("자치구명 목록:")
    for gu in gu_names:
        print(gu)
    ```
    ```python
    caregiver_by_gu = caregiver_df.groupby('자치구명')['인원(명)'].sum().reset_index()
    caregiver_by_gu.rename(columns={'자치구명': '자치구', '인원(명)': '요양보호사 수'}, inplace=True)

    # 기존 df에 자치구 기준으로 병합
    df = pd.merge(df, caregiver_by_gu, on='자치구', how='left')

    df
    ```
    ```python
    # 요양보호사 수의 평균으로 중랑구 결측치 채우기
    mean_caregiver = df['요양보호사 수'].mean()
    df.loc[df['자치구'] == '중랑구', '요양보호사 수'] = mean_caregiver
    ```
    (5) 정류장 수 & 평균 노선 수: 그룹 통계 계산 후 병합
    ```python
    bus_stop_df = pd.read_excel("/content/drive/MyDrive/코랩/데이터마이닝/팀플/data/서울시 정류소현황(2019~2023년).xlsx")

    # 자치구별로 정류장 수 (고유 ARS-ID 개수)와 평균 노선 수 계산
    bus_stats = bus_stop_df.groupby('행정구명').agg(
    정류장수=('ARS-ID', 'nunique'),
    평균노선수=('노선수', 'mean')
    ).reset_index()

    # 열 이름 정리
    bus_stats.rename(columns={'행정구명': '자치구'}, inplace=True)


    # 기존 df에 병합
    df = pd.merge(df, bus_stats, on='자치구', how='left')

    df
    ```
    (6) 노인 보행 교통사고: 최신 통계값 병합
    ```python
    traffic_data = pd.read_excel('/content/drive/MyDrive/코랩/데이터마이닝/팀플/data/서울시 노인 교통사고 현황 통계.xlsx', sheet_name='데이터')

    # 필요한 열만 선택
    traffic_data_selected = traffic_data[['자치구', '노인 보행 교통사고']]

    df = df.merge(traffic_data_selected, on='자치구', how='left')

    df
    ```
- 데이터 정제
    - 불필요한 열 제거 및 열 이름 통일
    - 문자형 → 수치형 변환
    - 결측치(NaN) → 0 또는 평균값으로 대체
- 최종적으로 분석에 사용한 전처리 데이터
- <img src="https://github.com/user-attachments/assets/8b6efd1d-5b97-4e59-913e-575f9ea6cc3d" width="400" height="350"/>

## 4. EDA
- boxplot : 변수들의 이상치 및 분포 확인
<pre>
전체 변수에 대해 boxplot을 이용한 시각화를 수행하였다. 이를 통해 변수 간 분포 범위, 이상치(outlier)의 존재 여부, 데이터 스케일 차이 등을 직관적으로 파악할 수 있었다.

- 독거노인 수, 요양보호사 수, 정류장 수 등에서 극단값(outlier)이 존재
- 자살률, 복지관 수, 평균 노선 수 등은 상대적으로 좁은 범위를 가짐
- 변수 간 스케일이 다양하므로 이후 분석에서는 Z-score 표준화가 필수적임
    </pre>
![image](시각화 자료 : 각 변수 별 boxplot)
- 변수 간 특성 차이 시각화 및 이상치 탐색
![image](시각화 자료 : 변수 간 분포 비교)

## 5. k-means 클러스터링
<pre>
(1) 주요 변수들을 노인 인구 수로 나누어 1인당 기준으로 환산
(2) StandardScaler를 활용해 모든 수치형 변수 정규화
(3) k=2~10까지의 경우를 PCA 분석을 통해 2차원으로 축소하여 시각화 및 클러스터링 수행
<img src="https://github.com/user-attachments/assets/32f0cbf0-df83-42b2-afc5-b7862dcd4c88" width="500" height="350"/>
(4) 2차원 PCA결과에 대해 k=2~6까지의 실루엣 계수와 엘보우 분석을 통해 최적 k선정(k=3)
(5) 최종적으로 k=3으로 클러스터링 수행 후 k=2~6까지의 경우를 시각화를 통해 지역 분포 확인
<img src="https://github.com/user-attachments/assets/38c527cb-f2b9-47cc-aed0-a12c2e30a287" width="500" height="350"/)
(6) Cluster 1은 다른 클러스터들에 비해 내부 특성 차이가 크고, 정책적으로도 세분화가 필요한 지역들이 포함되어 있어 Cluster 1의 세부 클러스터링 진행
(7) Elbow & Silhouette 분석 진행 결과, Cluster 1도 3개의 하위 그룹으로 나누는 것이 적절
<img src="https://github.com/user-attachments/assets/02920aaa-5e98-42b3-9076-ee9f0b305de0" width="650" height="350"/>
(8) Cluster 1안에서도 서로 다른 복지 특성을 가지는 자치구들이 존재함을 지도 시각화를 통해 확인
<img src="https://github.com/user-attachments/assets/d8998e84-c85f-4143-96da-87d163d8d0d9" width="400" height="300")
    </pre>

## 6. 클러스터링 해석 및 시각화
- Cluster 1을 k=3로 재클러스터링한 Sub-cluster 0~2 해석
<pre>
    (1) Sub-cluster 0 (광진구, 동대문구, 중랑구, 성북구, 강북구, 도봉구, 노원구, 은평구, 양천구, 구서구, 구로구, 관악구, 송파구, 강동구) (수정)
        - 노인 인구 비중 다소 적고 노인 보행 교통사고 비중이 매우 작음
        - 경로당 수 거의 없음
        - 요양보호사 수 비교적 많아서 복지 서비스 제공 여건이 좋음 시사
    (2) Sub-cluster 1 (용산구, 성동구, 서대문구, 마포구, 영등포구, 동작구, 서초구, 강남구) (ppt수정)
        - 노인인구 비중이 가장 높고 요양보호사 수 비중이 다소 낮음
        - 복지 취약 가능성 높음
    (3) Sub-cluster 2 (종로구, 중구, 금천구) (ppt수정)
        - 노인 인구는 많지만, 요양보호사 비중도 상대적으로 높음
    </pre>
![image](시각화 자료 : Sub-cluster0~2 파이차트)
- 대표 자치구 선정 (수정)
    - 각 클러스터의 평균값을 구하고 평균값에 가장 가까운 자치구를 대표로 지정함
    <pre>
    (1) Sub-cluster 0 : 광진구
    (2) Sub-cluster 1 : 용산구
    (3) Sub-cluster 2 : 종로구
        </pre>
![image](시각화 자료 : 각 클러스터의 평균값 표)
![image](시각화 자료 : 각 클러스터의 대표값 표시 지도)
- 클러스터별 변수 평균 시각적으로 확인
<pre>
    (1) Sub-cluster 1
        - 노인인구 비중이 가장 높고 요양보호사 수 비중이 다소 낮음
        - 복지 취약 가능성 높음
    (2) Sub-cluster 2
        - 노인 인구는 많지만, 요양보호사 비중도 상대적으로 높음
    </pre>
![image](시각화 자료 : 클러스터별 변수 평균)
- 차이가 큰 변수들 표준화 후 시각적으로 비교
<pre>
    (1) Sub-cluster 0 : 요양 보호사 수, 노인보행교통사고, 노인 인구 수 모두 Z-score 양수
        -> 노인 인구, 돌봄 인력 많지만 사고도 많은 지역
    (2) Sub-cluster 1 : 노인 보행 교통 사고 Z-score가 가장 낮음 (음수), 전반적으로 Z-score 0 근처로 큰 특이점 없음
        -> 균형 잡힌 평균형 지역 (ppt수정)
    (3) Sub-cluster 2 : 모든 변수에서 Z-score가 음수로 평균보다 낮음 
        -> 노인 인구 적고 인프라도 부족, 사고가 적지만 전반적으로 취약 (ppt 수정)
    </pre>

## 7. 정책 우선순위 도출
- 변수 카테고리 : 인프라, 서비스, 이동성, 취약성
- 변수별 가중치 설정 : 전문가가 평가한 정책별 중요도 평가를 통해서 가중치 설정함
```python
#  카테고리별 변수 지정
infra_vars = ['1인당 복지관 수', '1인당 노인의료복지시설 수', '1인당 재가노인복지시설 수']
service_vars = ['요양보호사 수', '1인당 복지관종사자 수']
mobility_vars = ['1인당 정류장수', '평균노선수']
vulnerable_vars = ['1인당 독거노인 합계', '추정 노인 자살률']

#  Z-score로 표준화
standardized = (cluster_summary - cluster_summary.mean()) / cluster_summary.std()

# 가중치 설정
weights = {
    'infra': 0.15,
    'service': 0.25,
    'mobility': 0.15,
    'vulnerable': 0.45
}
```
![image](시각화 자료 : 정책 영역별 중요성)
```python
# 카테고리별 평균 점수 (취약계층 역방향)
infra_score = standardized[infra_vars].mean(axis=1)
service_score = standardized[service_vars].mean(axis=1)
mobility_score = standardized[mobility_vars].mean(axis=1)
vulnerable_score = -standardized[vulnerable_vars].mean(axis=1)

# 가중합으로 최종 우선순위 점수 계산
priority_score = (
    infra_score * weights['infra'] +
    service_score * weights['service'] +
    mobility_score * weights['mobility'] +
    vulnerable_score * weights['vulnerable']
)

# 결과 데이터프레임
priority_df = pd.DataFrame({
    'Priority Score': priority_score
}, index=[0, 1, 2])

# 자치구-클러스터 매핑
gu_cluster_df = pd.DataFrame({
    '자치구': ['종로구', '중구', '용산구', '성동구', '광진구', '동대문구', '중랑구', '성북구', '강북구', '도봉구',
             '노원구', '은평구', '서대문구', '마포구', '양천구', '강서구', '구로구', '금천구', '영등포구', '동작구',
             '관악구', '서초구', '강남구', '송파구', '강동구'],
    '클러스터_PCA_3': [2, 2, 1, 1, 0, 0, 0, 0, 0, 0,
                    0, 0, 1, 1, 0, 0, 0, 2, 1, 1,
                    0, 1, 1, 0, 0]
})

# 클러스터별 자치구명 리스트로 정리
cluster_gu_dict = gu_cluster_df.groupby('클러스터_PCA_3')['자치구'].apply(list).to_dict()

# 우선순위 높은 순으로 정렬
priority_df_sorted = priority_df.sort_values('Priority Score', ascending=True)
priority_df_sorted['정책 도입 순위'] = range(1, len(priority_df_sorted) + 1)

# 각 순위별로 포함된 자치구 출력
print(" 클러스터별 최종 정책 도입 순위 및 자치구:")
for idx, row in priority_df_sorted.iterrows():
    cluster_id = idx
    gu_list = cluster_gu_dict.get(cluster_id, [])
    순위 = int(row['정책 도입 순위'])
    print(f"\n순위 {순위}위 (클러스터 {cluster_id}) - 정책 시급도: {row['Priority Score']:.3f}")
    print("포함된 자치구:", ', '.join(gu_list))
```
- 클러스터별 종합점수 계산 바탕으로 우선순위 설정
- 최종 정책 도입 순위 및 자치구
    <pre>
    1순위 : Sub-cluster 2 , 정책 시급도 : -0.190
    포함된 자치구: 종로구, 중구, 금천구
    2순위 : Sub-cluster 0 , 정책 시급도 : -0.018
    포함된 자치구: 광진구, 동대문구, 중랑구, 성북구, 강북구, 도봉구, 노원구, 은평구, 양천구, 구서구, 구로구, 관악구, 송파구, 강동구
    3순위 : Sub-cluster 1 , 정책 시급도 : 0.208
    포함된 자치구 : 용산구, 성동구, 서대문구, 마포구, 영등포구, 동작구, 서초구, 강남구
    </pre>
- 실제 종로구는 노인 복지 사각지대 문제 진행 중
    -> 분석 결과 신뢰할만 함
![image](사진 자료 : 뉴스 이미지 삽입)

## 8. 결론 및 정책 제안
(1) cluster0
<pre>
시설 중심에서 커뮤니티 중심으로 전환
모빌리티 접근성 보완
</pre>
(2) cluster1
<pre>
현행 수준 유지 + 선도 모델 전환
</pre>
(3) cluster2
<pre>
요양보호사 및 복지 인력 확충
고립노인 예방 프로그램
자살 예방 및 정신건강 복지 강화
</pre>

## 9. 한계점 및 향후 개선 방안
- 한계점 1. 변수 구성 제약
<pre>
    - 노인 자살률 데이터 확보가 어려워 추정된 데이터 사용함
    - 중랑구 요양 보호사 수를 정확히 알 수 없음
    - 정량적 수치 기반으로 정성적 요소 반영하지 못 함
=> 개선 방안
    - 더욱 자세한 데이터 확보
    - 노인 대상 설문조사 및 인터뷰 결과를 정성 변수로 포함
</pre>
- 한계점 2. 특정 대규모 자치구 영향력 과대 가능성
<pre>
    - 노인 인구 수가 많은 자치구가 클러스터 중심에 크게 영향을 줄 수 있어 중소 자치구 특성이 묻히는 문제 발생 가능성 있음
=> 개선 방안
    - 경우에 따라 가중치 없는 거리 기반 군집화 방식 도입 고려
</pre>
- 한계점 3. 데이터 최신성 및 범위의 제한
<pre>
    - 2023년도 기준 데이터 활용으로 데이터 최신성이 떨어짐
    - 분석 지역이 서울시로 한정되어 있어 일반화에 제약 있음
=> 개선 방안
    - 최신 연도 기준 데이터로 주기적 업데이트 진행
    - 서울 외 타 광역시 또는 전국 단위로 지역 비교 분석 확대
</pre>

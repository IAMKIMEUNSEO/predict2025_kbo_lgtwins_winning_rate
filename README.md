# 2025년 엘지트윈스 경기 승률 예측
엘지트윈스 팬 경험의 질적 향상을 위한 데이터 기반 분석 프로젝트트

## 1. 프로젝트 개요
- 배경
    - 경기 결과의 예측 불가능성으로 인한 관람 만족도 저하
    - 데이터 활용을 통한 전략적 예매로 시간과 비용의 효율성 제고
- 목표
    - 엘지트윈의 경기 승률에 영향을 미치는 요인 파악악
    - 2025년 엘지트윈스의 상대 구단에 따른 구장(홈·원정)별·시리즈(주중·주말)별 경기 승률
- 기대효과
    - 팬의 합리적인 관람 결정을 위한 정보 제공
    - 경기 관람 만족도와 팀 응원 몰입도 향상
    - 경기 전략 개선을 위한 데이터 활용용
- 일정: 2025년 3월 14일(금) ~ 2025년 3월 20일(목)
- 사용 언어: Python 3.11.9

## 2. 승률 예측에 사용할 변수 선정
- **승률 컬럼인 'winning_rate' 생성**
    - `df_kia['winning_rate'] = (df_kia['W'] / df_kia['GS'] *100).round(2)`
- **승률에 영향을 미치는 주요 특성 파악**
    - **`RandomForestRegressor`**를 활용하여 변수 중요도를 분석
    - 독립변수: `df_team.loc[:, 'S':'ERA']`
    - 종속변수: `df_team['winning_rate']`
    - 특성 중요도 상위 15개 확인
    ```python
    importances = rf.feature_importances_
    feature_names = train_input.columns

    #랜덤 포레스트 변수 중요도
    importance_df = pd.DataFrame({'Feature': feature_names, 'Importance': importances})
    importance_df = importance_df.sort_values(by='Importance', ascending=False)

    #상위 15개 선택
    selected_features = importance_df['Feature'].head(15).tolist()
    selected_features   #RS, OPS, ERA, RS9, AVG, R, S, IP, WHIP, ER, HR, SO, H, SF, OBP
    ```
    - 상위 15개 특성 투수/타자 항목으로 분류
        - 투타조화 고려
        - 각각 상위 5개를 선택해서 승률에 가장 영향을 미치는 요인으로 선정

|분류|지표|
|:---:|---|
투수|RS(득점지원), RS9(9이닝당 득점지원), R(득점), HR(홈런), H(안타), SF(희생플라이)|
|타자|OPS(피OPS), ERA(평균자책점), AVG(피안타율), S(홀드), IP(이닝), WHIP(이닝당 출루 허용률), ER(자책점), SO(삼진), OBP(피출루율)|

## 3. 구장(홈·원정)별 승률
### (1) 승률 예측 과정
- **승률에 영향을 미치는 투타지표 예측**
    - 구단에 따른 구장별 기록을 웹스크롤링해서 데이터프레임으로 변환한 후, 홈과 원정으로 데이터프레임을 분리
    ```python
    df_kiaH = df_kia.iloc[:10, 1:]  #홈
    df_kiaA = df_kia.iloc[10:, 1:]  #원정
    ```
    - 독립변수: G, GS, W, L
    - 종속변수: 선정한 투타 지표 10개(RS, RS9, R, HR, H, OPS, ERA, AVG, S, IP)
    ```python
    game = np.array(df_kiaH[['G', 'GS', 'W', 'L']]) #(출장, 게임, 승, 패)
    record = np.array(df_kiaH.iloc[:, :-5]) #기록('RS', 'RS9', 'R', 'HR', 'H', 'OPS', 'ERA', 'AVG', 'S', 'IP')

    x_game = game   #게임수
    y_record = record   #기록('RS', 'RS9', 'R', 'HR', 'H', 'OPS', 'ERA', 'AVG', 'S', 'IP')
    ```
    - <b>`LinearRegression`</b>을 활용하여 모델을 생성하고 학습
    ```python
    #기록 예측 모델
    model_record = LinearRegression()   #생성
    model_record.fit(x_game, y_record)   #학습
    ```
    - 독립변수의 평균값으로 2015년 기록을 예측
    ```python
    #2025년 기록 예측
    pred_record = model_record.predict([df_kiaH[['G', 'GS', 'W', 'L']].mean().values])[0]  #평균
    pred_RS, pred_RS9, pred_R, pred_HR, pred_H, pred_OPS, pred_ERA, pred_AVG, pred_S, pred_IP = pred_record
    ```
- **2025년 구장별 승률 파악**
    - 독립변수: 선정한 투타지표 10개 (RS, RS9, HR, H, OPS, ERA, AVG, S, IP)
    - 종속변수: winning_rate
    ```python
    winning_rate = np.array(df_kiaH['winning_rate']) #승률

    x = record  #RS, RS9, HR, H, OPS, ERA, AVG, S, IP
    y = winning_rate    #승률
    ```
    - <b>`LinearRegression`</b>을 활용하여 모델을 생성하고 학습
    ```python
    model_winning_rate = LinearRegression() #생성
    model_winning_rate.fit(x, y)    #학습
    ```
    - 위에서 예측한 2025년 기록으로 2025년 승률 예측
    ```python
    #예측 데이터
    pred_x = np.array([[
        pred_RS, pred_RS9, pred_R, pred_HR, pred_H,
        pred_OPS, pred_ERA, pred_AVG, pred_S, pred_IP
    ]])

    #승률 예측
    predicted_kiaH = model_winning_rate.predict(pred_x)[0].round(2)
    print(f"2025년 승률 예측: {predicted_kiaH:}")   #2025년 승률 예측: 55.08
    ```

### (2) 분석
- 야구는 결과(승/패)가 중요함으로 50%를 기준으로 예측 승률 분석

|그래프|분석|
|:---:|---|
|![2025년 구단별 홈과 원정 승률 예측 비교](/graph_img/bar_lgtwinsHome&Away2025WinRatePrediction.png)|**홈 승률(50% 기준)**<br>    - 평균 54.35%<br>   - ↑: 케이티(61.59%), 삼성(57.88%), 롯데(57.38%), 키움(56.92%), 한화(56.09%), 기아(56.09%), 쓱(52.56%), 엔씨(51.75%)<br> - ↓: 두산(39.88%)<br><br>**원정 승률(50% 기준)**<br>  - 평균: 46.98%<br>  - ↑: 케이티(56.45%), 한화(56.27%), 엔씨(50.16%), 키움(50.12%)<br>   - ↓: 롯데(45.99%), 쓱(46.17%), 기아(46.98%), 두산(47.70%), 삼성(48.55%)<br><br>**구장별 승률 차이**<br>  - 홈↑: 롯데(11.39%), 삼성(9.33%), 기아(8.10%), 키움(6.80%), 쓱(6.39%), 케이티(5.14%), 엔씨(1.59%)<br>   - 원정↑: 두산(-7.82%), 한화(-0.18%)<br> - 전체적으로 원정보다 홈 승률이 높은 경우가 더 많음<br> - 절댓값: 롯데(11.39%), 삼성(9.33%), 기아(8.10%), 두산(7.82%), 키움(6.80%), 쓱(6.39%), 케이티(5.14%), 엔씨(1.59%), 한화(0.18%)|

## 4. 시리즈(주중·주말)별 승률
### (1) 승률 예측 과정
- **승률에 영향을 미치는 투타지표 예측**
    - 구단에 따른 구장별 기록을 웹스크롤링해서 데이터프레임으로 변환한 후, 홈과 원정으로 데이터프레임을 분리
    ```python
    df_kia_sun = df_kia_day.iloc[:10, 1:]    #일요일
    df_kia_tue = df_kia_day.iloc[10:20, 1:]  #화요일
    df_kia_wed = df_kia_day.iloc[20:30, 1:]  #수요일
    df_kia_thu = df_kia_day.iloc[30:38, 1:]  #목요일
    df_kia_fri = df_kia_day.iloc[38:47, 1:]  #금요일
    df_kia_sat = df_kia_day.iloc[47:, 1:]    #토요일

    df_kia_weekdays = pd.concat([df_kia_tue, df_kia_wed, df_kia_thu], axis=0)   #주중 시리즈 결합
    df_kia_weekends = pd.concat([df_kia_fri, df_kia_sat, df_kia_sun], axis=0)   #주말 시리즈 결합
    ```
    - 독립변수: G, GS, W, L
    - 종속변수: 선정한 투타 지표 10개(RS, RS9, R, HR, H, OPS, ERA, AVG, S, IP)
    ```python
    game = np.array(df_kia_weekdays[['G', 'GS', 'W', 'L']]) #(출장, 게임, 승, 패)
    record = np.array(df_kia_weekdays.iloc[:, :-5]) #기록('RS', 'RS9', 'R', 'HR', 'H', 'OPS', 'ERA', 'AVG', 'S', 'IP')

    x_game = game   #게임수
    y_record = record   #기록('RS', 'RS9', 'R', 'HR', 'H', 'OPS', 'ERA', 'AVG', 'S', 'IP')
    ```
    - <b>`LinearRegression`</b>을 활용하여 모델을 생성하고 학습
    ```python
    #기록 예측 모델
    model_record = LinearRegression()   #생성
    model_record.fit(x_game, y_record)   #학습
    ```
    - 독립변수의 평균값으로 2015년 기록을 예측
    ```python
    #2025년 기록 예측
    pred_record = model_record.predict([df_kia_weekdays[['G', 'GS', 'W', 'L']].mean().values])[0]  #평균
    pred_RS, pred_RS9, pred_R, pred_HR, pred_H, pred_OPS, pred_ERA, pred_AVG, pred_S, pred_IP = pred_record
    ```
- **2025년 시리즈별 승률 파악**
    - 독립변수: 선정한 투타지표 10개 (RS, RS9, HR, H, OPS, ERA, AVG, S, IP)
    - 종속변수: winning_rate
    ```python
    winning_rate = np.array(df_kia_weekdays['winning_rate']) #승률

    x = record  #RS, RS9, HR, H, OPS, ERA, AVG, S, IP
    y = winning_rate    #승률
    ```
    - <b>`LinearRegression`</b>을 활용하여 모델을 생성하고 학습
    ```python
    model_winning_rate = LinearRegression() #생성
    model_winning_rate.fit(x, y)    #학습
    ```
    - 위에서 예측한 2025년 기록으로 2025년 승률 예측
    ```python
    #예측 데이터
    pred_x = np.array([[
        pred_RS, pred_RS9, pred_R, pred_HR, pred_H,
        pred_OPS, pred_ERA, pred_AVG, pred_S, pred_IP
    ]])

    #승률 예측
    predicted_kia_weekdays = model_winning_rate.predict(pred_x)[0].round(2)
    print(f"2025년 승률 예측: {predicted_kia_weekdays:}")   #2025년 승률 예측: 49.11
    ```

### (2) 분석
- 야구는 결과(승/패)가 중요함으로 50%를 기준으로 예측 승률 분석

|그래프|분석|
|:---:|---|
|![2025년 구단별 홈과 원정 승률 예측 비교](/graph_img/bar_lgtwinsSeries2025WinRatePrediction.png)|**홈 승률(50% 기준)**<br>    - 평균 52.46%<br>   - ↑: 엔씨(61.73%), 케이티(61.55%), 한화(56.55%), 롯데(54.17%), 삼성(51.61%), 쓱(50.86%)<br>- ↓: 두산(36.86%), 기아(49.11%), 키움(49.7%)<br><br>**원정 승률(50% 기준)**<br>  - 평균: 50.76%<br>  - ↑: 삼성(57.37%), 키움(57.36%), 케이티(53.84%), 한화(51.14%), 기아(50.14%)<br>   - ↓: 두산(44.19%), 엔씨(46.92%), 롯데(47.99%), 쓱(47.99%)<br><br>**시리즈별 승률 차이**<br>  - 주중↑: 쓱(2.87%), 한화(5.41%), 롯데(6.32%), 케이티(7.71%), 엔씨(14.81%)<br>   주말↑: 키움(-7.65%), 두산(-7.33%), 삼성(-5.76%), 기아(-1.03%)<br> - 시리즈별 승률 차이 개수는 비슷<br> - 절댓값: 엔씨(14.81%), 케이티(7.71%), 키움(5.76%), 한화(5.41%), 쓱(2.87%), 기아(1.03%)|

## 6. 결론 및 제언
|구단|분석 내용|
|:---:|---|
|**기아**|- 홈 승률(56.09%)은 평균(54.35%) 이상이지만 원정 승률(46.98%)은 세 번째로 낮으며, 둘의 차이는 8.10%로 세 번째로 가장 많이 차이 남<br>- 주중(49.11%)과 주말(50.14%) 승률 차이는 1.03%로 주중이 더 높으며 9개 구단 중 가장 차이가 안 남<br>- 홈과 주말은 기준치 이상이고 원정과 주중은 기준치 미달이므로 **주말 홈 경기**가 가장 최상의 조합|
|**삼성**|- 홈 승률(57.88%)은 9개 구단 중 가장 높지만 원정 승률(48.55%)은 기준치인 50%보다 낮으며, 둘의 차이는 9.33%로 두 번째로 가장 차이 많이 남<br>- 주중 승률은 51.61%이고, 주말 승률은 57.37%로 주말 승률은 9개 구단 중 가장 높지만 두 번째로 높은 키움과 0.01%밖에 차이 안 남<br>- 홈은 기준치 이상이지만 원정은 기준치 미달이며 시리즈 승률은 모두 기준치 이상이지만 시리즈 승률 차이를 고려하면 **주말 홈 경기**가 가장 최상의 조합|
|**두산**|- 홈 승률(39.88%)은 가장 낮으며 두 번째로 낮은 엔씨와는 11.87% 차이 남<br>- 홈(39.88%)과 원정(47.70%) 승률 차이는 7.82%로 홈 승률이 더 높음<br>- 주중(36.86%)과 주말(44.19%) 승률 모두 9개 구단 중 가장 낮으며, 주중 승률은 두 번째로 낮은 기아와는 12.25% 차이 남<br>- 모든 승률이 기준치 미달이므로 **두산과의 경기는 피하는 쪽으로 고려**하는 것이 좋음|
|**쓱**|- 홈 승률은 52.86%이고 원정 승률은 46.17%로 원정 승률은 9개 구단 중 두 번째로 가장 낮음<br>- 주중(50.86%)과 주말(47.99%) 승률 차이는 2.87%로 두 번째로 가장 차이가 안 남<br>- 홈과 주중 승률은 기준치 이상이지만 원정과 주말은 기준치 미달로 **주중 홈 경기**가 가장 최상의 조합|
|**롯데**|- 홈 승률(57.38%)은 세 번째로 가장 높고 원정 승률(45.99%)은 가장 낮고 둘의 승률 차이는 11.39%로 9개 구단 중 가장 높음<br>- 주중(54.17%)과 주말(47.99%) 승률 차이는 6.32%이며 주말 승률은 세 번째로 가장 낮음<br>- 홈과 주중은 기준치 이상이지만 원정과 주말은 기준치 미달이므로 **주중 홈 경기**가 가장 최상의 조합|
|**한화**|- 홈(56.09%)과 원정(56.27%) 승률 차이는 0.18%로 9개 구단 중 가장 적게 차이나며 둘의 큰 차이 또한 별로 없음<br>- 주중(56.55%)과 주말(51.14%) 승률 차이는 5.41%로 3번째로 가장 차이가 안 나며 주중 승률은 세 번째로 높음<br>- 모든 승률이 전부 기준치 이상이므로 모든 조합이 무난하지만 시리즈 승률 차이를 고려하면 홈과 원정 상관없이 **주중 시리즈**가 가장 최상의 조합|
|**엔씨**|- 홈 승률(51.75%)은 두 번째로 낮고 원정 승률(50.16%)은 세 번째로 높으며 둘의 차이는 1.59%로 두 번째로 낮음<br>- 주중 승률(61.73%)은 가장 높고 주말 승률(46.92%)는 두 번째로 낮으며 둘의 차이는 14.81%로 압도적으로 가장 높음<br>- 홈, 원정, 주중 승률은 기준치 이상이지만 주말은 기준치 미달이므로 홈과 원정 상관없이 **주중 시리즈**가 가장 최상의 조합|
|**케이티**|- 홈(61.59%)과 원정(56.45%) 승률 모두 9개 구단 중 가장 높으며, 홈 승률은 유일하게 60%대 수치를 기록했고, 둘의 승률 차이는 5.14%로 3번째로 낮음<br>- 주중 승률(61.55%)은 두 번째로 높고 주말 승률(53.84%)은 세 번째로 높으며 둘의 차이는 7.71%로 두 번째로 높음<br>- 유일하게 60%대 수치를 두 번이나 기록<br>- 모든 승률이 전부 기준치 이상이므로 모든 조합이 무난하지만 60% 수치를 기록한 **주중 홈 경기**가 가장 최상의 조합|
|**키움**|- 홈 승률은 56.92%이고 원정 승률은 50.12%이며 둘의 차이는 6.80%<br>- 주중 승률(49.71%)은 세 번째로 낮고 주말 승률(57.36%)은 두 번째로 높으며 둘의 차이는 7.65%로 세 번째로 높음<br>- 주중 승률만 기준치 미달이고 홈과 원정 승률 차이를 고려하면 **주중 홈 경기**가 가장 최상의 조합|




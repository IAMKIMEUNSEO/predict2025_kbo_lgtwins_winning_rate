# 2025년 엘지트윈스 경기 승률 예측

## 1. 프로젝트 개요
### (1) 프로젝트 배경
|   항목    |   내용    |
|:---:|---|
|팬 경험 확대|스포츠 관람 문화의 발전에 따라, 단순 관람을 넘어 더 풍부한 경험 추구|
|경기 결과 변수|예측 불가능성으로 인해 투자한 시간과 비용 대비 만족도 저하 가능성|
|데이터 활용|엘지트윈스의 승률 패턴 분석으로 전략적으로 예매|

### (2) 목표
- 엘지트윈스의 경기 승률에 영향을 미치는 요인
- 2025년 엘지트윈스 승률 예측
- 2025년 엘지트윈스의 상대 구단에 따른 구장(홈·원정)별·(주중·주말)별 경기 승률

### (3) 일정
- 2025년 3월 14일(금) ~ 2025년 3월 20일(목)

## 2. 데이터 수집 및 분석
### (1) 데이터 출처 및 수집 방법
- 출처: 스포키 STATIZ 팀 기록실 (엘지트윈스)
- 수집 방법: 웹 스크롤링 기법을 활용하여 2015년부터 2024년까지 엘지트윈스의 기록 데이터를 수집

### (2) 데이터 개요
- 데이터 항목
    - Year, G, GS, W, L, S, HD, IP, R, ER, RS, RS9, TBF, AB, H, 2B, 3B, HR, BB, IB, HP, SO, SF, NP, WHIP, AVG, OBP, OPS, ERA
- 전처리 계획
    - 중복 또는 불필요한 컬럼 제거
    - 컬럼 'GS'와 'W' 컬럼을 나눠 'winning_rate'란 새로운 컬럼 생성
- 기본 분석 방법
    - 경기 승률에 영향을 미치는 요인
    - 상대 구단에 따른 시리즈별 승률
    - 상대 구단에 따른 홈과 원정일 때의 승률

## 3. 기대효과 및 활용방안
### (1) 팬 입장
- 승리 가능성이 높은 경기 관람
    - 응원하는 팀이 승리하는 순간을 직접 경험할 가능성 높아짐으로써 큰 만족감을 느낌
- 한정된 예산 내에서 최적의 관람 기회 확보
    - 예매 비용 증가 등 물가 상승 속에서 경기 승률을 고려한 전략적 예매를 통해 가성비 있는 경기를 관람
- 관람 만족도 및 몰입감 향상
    - 유리한 경기 환경(특정 구장에서의 높은 승률, 특정 시리즈의 강세 등)을 고려하여 예매할 수 있어, 더욱 깊이 있는 응원과 감동을 경험
- 데이터 기반의 스마트한 야구 관람 문화 장착
    - 감(感) 또는 일정만 고려한 기존의 예매 방식에서 벗어나, 객관적인 데이터를 바탕으로 최적의 일정을 계획하는 문화를 조성

### (2) 구단 입장
- 팬 경험 증대 및 티켓 판매 전략 수립
    - 특정 시리즈 또는 구장에서의 승률 데이터를 활용해 팬들에게 보다 매력적인 경기 일정을 홍보하는 마케팅 전략을 수립
    - 예를들어, 승률이 높은 시리즈나 상대팀과의 경기를 중심으로 프로모션을 진행하거나 해당 경기의 관중 동원을 적극 유도함으로써 티켓 판매율 상승 기대
- 경기 전략 개선을 위한 데이터 활용
    - 경기 운영 전략 개선
    - 불리한 환경에서의 전술 변화, 특정 구장에서의 선수 전략 수정 등 다양한 방향으로 경기력 향상을 위한 참고 자료로 활용 가능
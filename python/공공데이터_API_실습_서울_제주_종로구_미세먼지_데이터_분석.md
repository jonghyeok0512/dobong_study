# 공공데이터 API 실습: 서울/제주/종로구 미세먼지 데이터 분석

### 1. 오늘 학습 요약
- API 호출의 기본 구조(URL, Params, Headers)와 인증키 관리법(.env) 습관화
- '지저분한 데이터'를 '사용 가능한 데이터'로 바꾸는 전처리(Cleaning) 과정 마스터
- 데이터가 많아질 때를 대비한 효율적인 자료구조(Dictionary) 활용법 학습

### 2. 배운 개념 정리
- **API Param 'ver'**: 같은 API라도 버전에 따라 응답 항목이 다름. 초미세먼지(PM2.5) 데이터를 받으려면 ver 1.0 이상 설정이 필수임.
- **Data Cleaning**: 외부 데이터는 언제나 비어있거나('-') 잘못된 타입이 올 수 있음. .isdigit() 등으로 먼저 검사해야 시스템 다운을 막음.
- **Dictionary Mapping**: 리스트에서 특정 데이터를 찾는 건 $O(N)$이지만, 딕셔너리로 미리 인덱싱해두면 $O(1)$로 즉시 접근 가능함.
- **HTTP Error 403**: 인증은 되었으나 권한이 없는 상태. API 활용 신청 승인 여부나 정확한 기능 URL을 확인해야 함.

### 3. 오늘 작성한 코드리뷰
```python
import requests
import os
from dotenv import load_dotenv

# 1. 보안을 위해 API 키는 환경변수에서 로드
load_dotenv()
AIR_KOREA_API_KEY = os.getenv('AIR_KOREA_API_KEY')
URL = '[http://apis.data.go.kr/B552584/ArpltnInforInqireSvc/getCtprvnRltmMesureDnsty](http://apis.data.go.kr/B552584/ArpltnInforInqireSvc/getCtprvnRltmMesureDnsty)'

# 2. 요청 파라미터 설정
# ver: 1.0을 넣어야 pm25Value가 포함됨
params = {
    'serviceKey': AIR_KOREA_API_KEY,
    'sidoName': '서울',
    'returnType': 'json',
    'ver': '1.0' 
}

# 3. 데이터 가져오기 및 예외 처리
response = requests.get(URL, params=params)
response.raise_for_status() # 403, 404 등 에러 발생 시 즉시 중단
data = response.json()
station_list = data['response']['body']['items']

# 4. 데이터 전처리 (거름망)
# - 값이 존재하고(get), 그 값이 숫자(isdigit)인 데이터만 추출
valid_stations = [
    item for item in station_list 
    if item.get('pm25Value') and item['pm25Value'].isdigit()
]

# 5. 자료구조 최적화 (이름으로 바로 찾기 위해 딕셔너리화)
# - 리스트를 순회하며 stationName을 키로 등록
station_mapping = { item['stationName']: item for item in valid_stations }

# 6. 안전한 데이터 접근 (방어적 프로그래밍)
target = '한강대로'
station_data = station_mapping.get(target) # 없으면 None 반환

if station_data:
    # 데이터가 있을 때만 인덱싱 접근
    print(f"[{target}] 농도: {station_data['pm25Value']}")
else:
    print(f"{target} 데이터가 존재하지 않습니다.")
```

### 4. 실수/에러와 해결 과정
- **NameError: name 'ver' is not defined**
    - **실수**: params 딕셔너리 정의 시 `'ver'`에 따옴표를 빼먹음.
    - **해결**: 파이썬은 따옴표가 없으면 변수로 인식하므로 문자열 처리를 정확히 해줌.
- **ValueError: invalid literal for int() with base 10: '-'**
    - **실수**: 측정소 점검 중 표시인 `-`를 강제로 int()로 바꾸려 함.
    - **해결**: .isdigit() 조건을 추가하여 숫자 형태의 문자열만 변환하도록 로직 수정.
- **TypeError: 'NoneType' object is not subscriptable**
    - **실수**: .get() 결과가 None인데 그 뒤에 바로 ['pm25Value']를 붙임.
    - **해결**: if문을 사용하여 데이터가 존재할 때만 내부 값에 접근하도록 수정.

### 5. 실무 관점의 참견 적용해보기
- **시간 복잡도**: 반복문 안에서 특정 값을 찾기 위해 또 반복문을 돌리는 실수(O(N^2))를 피하고, 딕셔너리 매핑(O(1))을 활용하여 수천 건의 데이터도 빠르게 처리하도록 설계함.
- **공간 복잡도**: 터미널에 대량의 데이터를 한 번에 출력(print)하면 시스템 부하가 발생할 수 있음을 체감함. 필요한 정보만 슬라이싱하거나 특정 키로 접근하여 출력하는 것이 효율적임.
- **견고함**: API 서버의 상태나 데이터 정합성을 믿지 않고, raise_for_status()와 데이터 클렌징 로직을 통해 어떤 상황에서도 프로그램이 죽지 않고 에러 메시지를 남기도록 구현함.
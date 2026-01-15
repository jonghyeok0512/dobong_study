# [복습 노트] 파이썬 객체지향 자판기 만들기 (Vending Machine Project)

## 1. 학습 목표 및 핵심 요약
이 프로젝트는 단순한 자판기 기능을 구현하는 것을 넘어, **"유지보수가 쉽고 확장이 가능한 코드"**를 짜는 방법을 익히는 데 목적이 있습니다.
* **객체 지향(OOP):** 데이터(상품 정보)와 기능(결제, 배출)을 묶어서 관리함.
* **다형성(Polymorphism):** '결제'라는 같은 명령어를 내려도 '카드'냐 '현금'이냐에 따라 다르게 작동하게 만듦.
* **데이터 접근:** 리스트 안에 들어있는 '객체 덩어리'를 다루는 법을 익힘.

---

## 2. 코드 상세 리뷰 (Line-by-Line 해석)

### Part 1: 상품(Product) 설계도
```python
class Product:
    # __init__: 객체가 처음 태어날 때(생성) 실행되는 약속
    # self는 '나 자신'을 의미함. 내 이름, 내 가격, 내 재고를 설정하는 과정.
    def __init__(self, name, price, stock):
        self.name = name    # 상품명 (예: "콜라")
        self.price = price  # 가격 (예: 1500)
        self.stock = stock  # 재고 (예: 5)

    # __str__: print(product)를 했을 때, 컴퓨터 언어가 아닌 사람이 보는 글자로 바꿔주는 역할
    def __str__(self):
        return f'"{self.name} : {self.price}원 (재고 : {self.stock}개)"'
```

### Part 2: 결제 시스템 (추상화와 상속)
이 부분은 **"규칙"**을 만드는 과정입니다. 나중에 '포인트 결제', '삼성페이'가 추가되어도 `process_payment`라는 이름만 똑같이 쓰면 자판기는 코드를 안 고쳐도 됩니다.

```python
from abc import ABC, abstractmethod

# Payment: 모든 결제 수단의 부모 (규칙 정의)
class Payment(ABC):
    @abstractmethod
    def process_payment(self, amount):
        pass # 자식들이 알아서 구현해라! (강제성 부여)

# CashPayment: 현금 결제 방식
class CashPayment(Payment):
    def __init__(self, received_amount):
        self.received_amount = received_amount # 손님이 투입한 돈

    def process_payment(self, amount):
        # 로직: 투입한 돈 >= 상품 가격
        if self.received_amount >= amount:
            return True
        return False

# CardPayment: 카드 결제 방식
class CardPayment(Payment):
    def __init__(self, card_limit):
        self.card_limit = card_limit # 카드 한도

    def process_payment(self, amount):
        # 로직: 카드 한도 >= 상품 가격
        if self.card_limit >= amount:
            return True
        return False
```

### Part 3: 자판기 본체 (핵심 로직)
가장 많은 고민과 실수가 있었던 부분. 객체 리스트를 순회하고 상태를 변경합니다.

```python
class VendingMachine:
    def __init__(self):
        # 자판기만의 창고 생성. 여기에는 문자열이 아니라 Product '객체'가 들어감.
        self.inventory = []

    def add_product(self, product):
        # 외부에서 만든 Product 객체를 내 창고(리스트)에 집어넣음
        self.inventory.append(product)
        
    def display_menu(self):
        # 창고에 있는 모든 객체를 하나씩 꺼내서 보여줌 (__str__ 덕분에 예쁘게 나옴)
        for product in self.inventory:
            print(product)
    
    # [중요] 상품 배출 및 결제 처리 로직
    def dispense_product(self, product_name, payment_method):
        
        # 1. 창고 탐색 (Search)
        for product in self.inventory:
            
            # [실수 포인트 1] 객체 vs 문자열 비교
            # if product == product_name: (X) -> 객체 덩어리와 글자를 비교하면 안 됨.
            # if product.name == product_name: (O) -> 객체 안의 '이름'을 꺼내서 비교해야 함.
            if product.name == product_name:
                
                # 2. 재고 확인 (Check Stock)
                # 찾아낸 그 객체(product)의 재고를 확인
                if product.stock > 0:
                    
                    # 3. 결제 시도 (Payment)
                    # payment_method가 카드인지 현금인지는 중요하지 않음.
                    # 그냥 process_payment 기능이 있다는 것만 믿고 맡김.
                    if payment_method.process_payment(product.price):
                        
                        # 4. 상태 업데이트 (Update State)
                        # 결제가 성공했으니 실제 객체의 재고를 1 줄임.
                        product.stock -= 1
                        print(f'{product.name}이(가) 배출되었습니다.')
                    
                    else:
                        # 결제 실패 (잔액 부족 등)
                        print('결제에 실패하였습니다.')
                
                else:
                    # 재고 없음
                    print('재고가 부족합니다.')
                
                # [실수 포인트 2] 함수의 종료 (Return)
                # 원하는 상품을 찾아서 처리(성공이든 실패든)를 했으면
                # 뒤에 남은 상품들을 더 볼 필요 없이 여기서 퇴근해야 함.
                # return이 없으면 아래 '상품을 찾을 수 없습니다'가 출력되는 버그 발생.
                return
        
        # for문을 다 돌았는데도 return을 만나지 못했다면? = 상품이 없다.
        print('상품을 찾을 수 없습니다.')
```

---

## 3. 내가 겪은 시행착오와 해결 (Troubleshooting)

### Q1. 왜 `product`와 `product_name`을 비교하면 안 되나?
* **현상:** `if product == "콜라":` 라고 썼더니 코드가 동작하지 않음.
* **원인:** `product`는 [이름, 가격, 재고]가 포장된 **선물 상자(객체)**이고, `"콜라"`는 **쪽지(문자열)**임. 상자와 쪽지는 당연히 다름.
* **해결:** 상자를 열어서 그 안의 이름표를 꺼내 비교해야 함. (`product.name == "콜라"`)

### Q2. `return`은 언제 쓰는가?
* **현상:** 상품을 샀는데도 마지막에 "상품을 찾을 수 없습니다"라는 문구가 또 뜸.
* **원인:** 상품을 찾고 나서 함수를 끝내지 않아서, 코드가 바닥까지 계속 실행됨.
* **해결:** "찾았으면 끝!"을 명시하기 위해 `return`을 사용하여 함수를 즉시 종료시킴.

---

## 4. 실무 개발자 관점 (Next Step)

### 1) 데이터베이스(DB)가 필요한 이유
지금 코드는 프로그램을 껐다 켜면 재고가 다시 초기화됩니다(콜라가 다시 5개가 됨).
* **실무:** 재고를 영구적으로 저장하기 위해 `MySQL` 같은 데이터베이스를 사용합니다. `self.inventory = []` 대신 `SELECT * FROM products` 쿼리를 쓰게 됩니다.

### 2) 로그(Log)의 중요성
지금은 `print()`로 화면에 띄우지만, 실제 자판기라면 "언제, 누가, 무엇을 샀는지" 기록해야 합니다.
* **적용:** `sales_log.append(f"{time.now()}: {product.name} 판매 완료")` 같은 코드를 추가하여 매출 데이터를 모을 수 있습니다.

### 3) RAG(검색 증강 생성)와 연결하기
사용자가 정확히 "콜라"라고 말하지 않고 **"톡 쏘는 검은 음료수 줘"**라고 했을 때 어떻게 할까요?
* **AI 적용:** 상품 설명에 "톡 쏘는 맛, 검은색" 같은 키워드를 넣어두고, 사용자의 말과 가장 비슷한 상품을 찾아주는(Vector Search) 기능을 붙이면 AI 자판기가 됩니다.
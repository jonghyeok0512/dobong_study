# [Project] 객체지향 기반 시립 통합 도서관 관리 시스템

## 1. 프로젝트 개요 및 학습 요약
본 프로젝트는 단순한 도서의 삽입/삭제 기능을 넘어, 실제 도서관 운영 환경에서 발생할 수 있는 **보안 문제, 데이터 무결성 문제, 다중 지점 연동 문제**를 해결하는 과정을 담고 있습니다.

## 2. 설계의 진화 과정 (사고의 흐름)

이 코드는 한 번에 완성된 것이 아니라, 문제를 발견하고 해결하는 과정에서 다음과 같이 발전했습니다.

### [Phase 1: 기능 중심의 초기 구현]
- **초기 상태**: 누구나 `add_book`, `remove_book`을 호출하면 책이 등록되고 삭제됨.
- **문제 발견**: "일반 회원이 마음대로 도서관 장부를 조작하면 안 되지 않나?"
- **해결책**: `Librarian` 클래스를 도입하고, `is_admin` 속성을 부여하여 관리자 여부를 구분하기 시작함.

### [Phase 2: 권한 검증의 일관성 확보]
- **중간 상태**: 삭제할 때는 관리자인지 확인하는데, 등록할 때는 확인하지 않는 코드가 작성됨.
- **문제 발견**: "삭제만큼 등록도 중요한 관리 행위다. 왜 여기는 검사를 안 하지?"
- **해결책**: `register_book` 메서드에서도 `requester`(요청자) 정보를 받아, 사서가 본인의 신분증(`self`)을 제시해야만 등록되도록 로직을 통일함.

### [Phase 3: 시립 통합 시스템으로의 확장]
- **확장 시도**: 강남점, 홍대점 등 도서관 객체를 여러 개 생성함.
- **치명적 오류**: 강남점에서 빌린 책을 홍대점에 반납했더니, 홍대점 장부에 없는 책인데도 반납 처리가 되어버림.
- **해결책 (데이터 무결성)**: 반납 로직에 `if book not in self.books` 조건을 추가하여, **"우리 지점 자산이 아니면 받지 않는다"**는 규칙을 세움.
- **정책 결정 (통합 한도)**: 지점은 여러 곳이지만 회원은 한 명이므로, `Member` 객체에 대출 목록을 몰아넣어 **"어디서 빌리든 총 3권까지만 가능"**한 시립 통합 대출 시스템을 완성함.

---

## 3. 전체 코드 리뷰 (Full Source Code Review)

작성된 전체 코드를 한 줄 한 줄 분석하며, 위에서 고민한 설계 철학이 어떻게 반영되었는지 주석으로 설명합니다.

```python
from abc import ABC, abstractmethod

# =========================================================
# [1] 도서(Book) 클래스
# 역할: 책의 기본 정보와 '상태(대출 가능 여부)'를 관리하는 데이터 객체
# =========================================================
class Book:
    def __init__(self, title, author, isbn: str):
        self.title = title
        self.author = author
        self.isbn = isbn
        # 책이 생성될 때 기본적으로 대출 가능(True) 상태로 시작
        self.is_available = True    
        # 실제 삭제 대신 플래그를 남기는 Soft Delete를 위한 준비 (확장성 고려)
        self.is_deleted = False     

    def borrow(self):
        # 대출 동작: 상태를 False로 변경
        self.is_available = False

    def return_book(self):
        # 반납 동작: 상태를 True로 복구
        self.is_available = True


# =========================================================
# [2] 사용자(User) 부모 클래스
# 역할: 회원과 사서가 공통으로 가질 속성(이름 등)을 정의
# =========================================================
class User:
    def __init__(self, name):
        self.name = name


# =========================================================
# [3] 회원(Member) 클래스
# 역할: 시립 도서관을 이용하는 고객. 통합 대출 목록을 소유함.
# =========================================================
class Member(User):
    def __init__(self, name, member_id):
        super().__init__(name)
        self.member_id = member_id
        # [핵심 설계] 회원의 대출 목록은 도서관이 아닌 회원이 들고 있음.
        # 이를 통해 A도서관에서 빌리든 B도서관에서 빌리든 이 리스트에 쌓이게 되어
        # '시립 통합 대출 한도' 관리가 가능해짐.
        self.borrowed_books = []    

    def add_book(self, book: Book):
        self.borrowed_books.append(book)
        print(f"[알림] {self.name}님의 통합 대출 목록에 '{book.title}' 추가됨.")

    def remove_book(self, book: Book):
        self.borrowed_books.remove(book)
        print(f"[알림] {self.name}님의 통합 대출 목록에서 '{book.title}' 삭제됨.")


# =========================================================
# [4] 사서(Librarian) 클래스
# 역할: 도서관 시스템을 조작할 수 있는 '권한(Authority)'을 가진 관리자
# =========================================================
class Librarian(User):
    def __init__(self, name, employee_id):
        super().__init__(name)
        self.employee_id = employee_id
        # [보안 핵심] 이 속성이 있어야만 도서관의 중요 기능(등록/삭제)에 접근 가능
        self.is_admin = True        

    def register_book(self, library_obj, book: Book):
        # [협력 패턴] 사서가 도서관 객체에게 명령을 내림.
        # 이때 self(자기 자신)를 인자로 넘겨주어 신분을 증명함.
        library_obj.add_book(self, book)
        print(f"[관리자 {self.name}] '{book.title}' 등록 요청 완료.")

    def remove_book(self, library_obj, book: Book):
        library_obj.remove_book(self, book)


# =========================================================
# [5] 도서관(Library) 클래스
# 역할: 실제 비즈니스 로직(대출/반납/검증)이 수행되는 핵심 장소
# =========================================================
class Library:
    def __init__(self, name):
        self.name = name            # 지점 이름 (예: 강남 도서관)
        self.books = []             # 지점별 보유 도서 리스트 (Inventory)
        self.LOAN_LIMIT = 3         # 정책: 1인당 최대 3권까지 대출 가능

    # --- [관리자 기능 구역] ---

    def add_book(self, requester, book):
        # [Guard Clause 1] 요청자가 관리자 권한이 있는지 확인
        # getattr를 사용해 is_admin 속성이 없거나 False면 즉시 차단
        if not getattr(requester, 'is_admin', False):
            print(f"권한 오류: {requester.name}님은 등록 권한이 없습니다.")
            return
        self.books.append(book)
        print(f"[{self.name}] 시스템: '{book.title}' 정상 등록됨.")

    def remove_book(self, requester, book):
        # [Guard Clause 1] 권한 체크
        if not getattr(requester, 'is_admin', False):
            print(f"권한 오류: 삭제 권한 없음.")
            return
        
        # [Guard Clause 2] 데이터 무결성 체크 (없는 책을 지울 순 없다)
        if book not in self.books:
            print(f"오류: 해당 도서는 {self.name}의 소장 자료가 아닙니다.")
            return
        
        # [Guard Clause 3] 상태 체크 (빌려간 책을 지우면 장부가 꼬인다)
        if not book.is_available:
            print(f"오류: 대출 중인 도서는 삭제할 수 없습니다.")
            return
        
        self.books.remove(book)
        print(f"[{self.name}] 시스템: '{book.title}' 삭제 완료.")

    # --- [사용자 기능 구역: 조회] ---

    def show_all_books(self):
        print(f"\n--- {self.name} 현재 보유 도서 ---")
        if not self.books:
            print("보유 도서 없음")
            return
        for book in self.books:
            status = "대출 가능" if book.is_available else "대출 중"
            print(f"- {book.title} | {status}")

    # --- [사용자 기능 구역: 트랜잭션(대출/반납)] ---

    def process_loan(self, member, book):
        # 1. 도서 상태 검증
        if not book.is_available:
            print(f"대출 불가: '{book.title}'은 이미 대출 중입니다.")
            return
        
        # 2. [통합 한도 정책 적용]
        # 해당 회원이 다른 도서관에서 빌린 책까지 합쳐서 3권이 넘는지 확인
        if len(member.borrowed_books) >= self.LOAN_LIMIT:
            print(f"대출 거부: {member.name}님은 통합 한도({self.LOAN_LIMIT}권) 초과.")
            return
        
        # 3. 모든 검증 통과 시 상태 변경
        book.borrow()           
        member.add_book(book)    
        print(f"대출 성공: {member.name} -> '{book.title}' ({self.name})")

    def process_return(self, member, book):
        # [데이터 무결성 핵심]
        # 타 도서관(예: 홍대) 책을 여기(강남)에 반납하려 할 때 차단해야 함.
        if book not in self.books:
            print(f"반납 오류: '{book.title}'은 {self.name}의 도서가 아닙니다.")
            return
        
        # 회원이 실제로 빌린 책인지 확인
        if book in member.borrowed_books:
            book.return_book()
            member.remove_book(book) 
            print(f"반납 성공: '{book.title}' 반납 완료.")
        else:
            print(f"반납 실패: 대출 목록에 존재하지 않음.")
```

## 4. 실무 관점의 배운 점 (Retrospective)
- **책임의 분리**: 도서관 객체는 '자산 관리'와 '정책 집행'을 담당하고, 사서 객체는 '행위의 주체'가 되며, 회원 객체는 '대출 상태'를 보유한다는 명확한 역할 분담이 코드의 복잡도를 낮춘다는 것을 배움.
- **방어적 프로그래밍**: 사용자가 항상 올바른 행동만 할 것이라고 믿지 않고, `process_return` 등에서 발생할 수 있는 이상한 요청(타 도서관 책 반납 등)을 코드로 사전에 차단하는 것이 시스템 안정성에 필수적임을 깨달음
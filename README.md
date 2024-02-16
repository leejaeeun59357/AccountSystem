## 개요
계좌(Account) 시스템 프로젝트

**Use** : Spring boot, Gradle, Junit5, H2 Database, JPA, Redis, Mockito, Lombok

**Goal** : 계좌를 생성하고 삭제 및 거래 후 잔액, 거래취소 잔액 등을 다루는 서버를 구축한다.

## 프로젝트 엔티티 구조
![image](https://github.com/leejaeeun59357/AccountSystem/assets/149572895/49fc6e56-7ae3-4215-a1e9-b1d7346451e8)


## API 명세서
### 1. 계좌(Account) 관련 API
#### 계좌 생성 API
- POST /account
- parameter : 사용자 아이디, 초기 잔액
- **결과**
  - **실패**
    - 사용자가 없는 경우
    - 사용자당 최대 보유 가능 계좌 10개를 넘어간 경우
  - **성공**
    - 계좌번호는 10자리 랜덤 숫자
    - 응답정보 : 사용자 아아디, 생성된 계좌 번호, 등록일시 (LocalDateTime)
  - **주의 사항**
    - 계좌 생성 시 계좌 번호는 10자리의 정수로 구성되며, 기존에 동일 계좌 번호가 있는지 중복체크를 해야 한다.
    - 기본적으로 계좌번호는 순차 증가 방식으로 생성한다.
   
- 저장이 필요한 정보

|칼럼명|데이터 타입|설명|
|:---|:---|:---|
|id|pk|primary Key|
|accountUser|AccountUser|소유자 정보, 사용자 테이블과 연결(n:1)|
|accountNumber|String|계좌 번호(자유도를 위해 문자열로 작성)|
|accountStatus|AccountStatus|계좌 상태(IN_USE, UNREGISTERED)|
|balance|Long|계좌 잔액|
|registeredAt|LocalDateTime|계좌 등록일시|
|unregisteredAt|LocalDateTime|계좌 해지일시|
|createdAt|LocalDateTime|생성일시|
|updatedAt|LocalDateTime|최종 수정일시|

#### 계좌 해지 API
- DELETE /account
- parameter : 사용자 아이디 계좌번호
- **결과**
  - **실패**
    - 사용자가 없는 경우
    - 계좌가 없는 경우
    - 아이디와 계좌 소유주가 다른 경우, 계좌가 이미 해지 상태인 경우, 잔액이 있는 경우
  - **성공**
      - 응답 ( 사용자 아이디, 계좌번호, 해지일시)<br/><br/>

#### 계좌 확인 API
- GET /account?user_id={userId}
- parameter : 사용자 아이디
  - **결과**
    - **실패**
       - 사용자 없는 경우
    - **성공**
        - List<계좌번호, 잔액> 구조로 응답

### 2. 거래(Transaction) 정보 관련 API
#### 잔액 사용 API
- POST /transaction/use
- parameter : 사용자 아이디, 계좌 번호, 거래 금액
  - **결과**
    - **실패**
        - 사용자 없는 경우
        - 사용자 아이디와 계좌 소유주가 다른 경우
        - 계좌가 이미 해지 상태인 경우
        - 거래 금액이 잔액보다 큰 경우
        - 거래 금액이 너무 작거나 큰 경우
    - **성공**
        - 응답(계좌번호, transaction_result(성공/실패), transaction_id, 거래금액, 거래일시)
- 저장이 필요한 정보

|칼럼명|데이터 타입|설명|
|:---|:---|:---|
|id|pk|Primary Key|
|transactionType|TransacitonType|거래의 종류(사용, 사용취소)|
|transactionResultType|TransactionResultType|거래 결과(성공, 실패)|
|account|Account|거래가 발생한 계좌(N:1 연결)
|amount|Long|거래 금액|
|balanceSnapshot|Long|거래 후 계좌 잔액|
|transactionId|String|계좌 해지 일시|
|transactedAt|LocalDateTime|거래 일시|
|createdAt|LocalDatetime|생성 일시|
|updatedAt|LocalDateTime|최종 수정 일시|


#### 잔액 사용 취소 API
- POST /transaction/cancel
- parameter : transaction_id, 계좌번호, 거래 금액
  - **결과**
    - 실패
        - 거래금액과 거래 취소 금액이 다른 경우 (부분 취소 불가능)
        - 트랜잭션이 해당 계좌의 거래가 아닌 경우
        - 1년이 넘은 거래일 경우 사용 취소 불가능
    - 성공
        - 응답 (계좌번호, transaction_result, transaction_id, 취소 거래금액, 거래일시)

#### 거래 확인 API
- GET /transaction/{transactionId}
- parameter : transaction_id
  - **결과**
    - **실패**
      - 해당 거래 아이디의 거래가 없는 경우
    - **성공**
        - 응답 (계좌번호, 거래종류(잔액 사용, 잔액 사용 취소), transaction_result(성공/실패), transaction_id, 거래일시, 거래금액)
    - 성공 거래 뿐만 아니라 실패 거래도 거래 확인할 수 있도록 코딩하기<br/>

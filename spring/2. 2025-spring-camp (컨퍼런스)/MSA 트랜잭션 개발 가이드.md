[ MSA 트랜잭션 개발 가이드 ]

- 마이크로서비스 아키텍쳐: 각팀이 독립적으로 일하는 것이 목적
- 다른 서비스의 테이블에 액세스하면 안됨
- 서비스간에 트랜잭션 보장이 안됨.. K8s, API, GW는 할테니 트랜잭션은 알아서 하세요?
> 분산 트랜잭션 FW 같은 것 > 사용하지 않는게 맞음
- msa > 원자성(db rollbakc), 독립성(read committed) 안됨
> 다른 서비스의 data를 가져올때 db 직접 < dao(repository) < servicelmpl(가장 모듈화, 대부분 좋음)

msa를 분리하는 법
- 속성으로 나누는것
- 레코드로 나누는 것

원자성(db rollback)
begin - write a - b - c - db
- 중간 쓰기 c에서 실패하면 db가 a,b 취소해줌
- 하지만 msa에서는 b서비스가 독립적이라 c에서 실패하면 a는 롤백되지만 b서비스는 커밋되버림

서비스간 트랜젹션은 보상 트랜잭션과 재시도

pivot transaction이 성공하면 성공한거고 실패하면 트랜잭션 취소, 재시도는 사용자가 주관
pivot transaction이후 실패하면 이벤트로 재시도, 계속 실패하면 담당자가 처리

보통 Pivot Transaction 내에서 실패하거나 성공하는 경우가 많음. (의미 있는 업무 > 서비스 업무)


트랜젝션 격리성(Isolation level 이 가장 자유로운, 낮은)
- 변경 중인 데이터를 다른 트랜잭션이 볼 수 있음
- 변경 중인 데이터를 다른 트랜잭션이 덮어 쓸 수 있음

이를 해결하기 위해
- Semantic Lock(영화 예매할때 예약중 시스템)
- TCC(Try, Confirm, Ca
ncel) > 커밋하는 단계를 try commit, confirm commit로 나눔
- Offline Lock


두 저장소에 쓰기의 어려움(
1. 원래 db의 생성, 다른 서비스의 api 호출 간 엇박
> db 생성과 이벤트 요청 생성을 하나의 트랜잭션으로 관리
2. 실패했는데 db는 성공한 경우
> 에러는 진짜 실패, 나머지는 가짜 실패일수도
ex, connectExeption 진짜 실패, Timeout 같은건 가짜 실패일수도.
- 가짜 실패 복구하는 법 > 한번 더 실행.
- 가짜 실패 했다는 정보조차 잃어버리면? > api 요청 하기 전에 AOP로 LOG로 먼저 저장 해두던지 해야함
- 멱등서 있게 구현하기 > DB에 유니크 값으로 식별, 사전 체크 OR 사후 에러 무시
- 전 이벤트가 ERROR나서 ERROR QUEUE에 빠져서 다음 이벤트 뒤로와서 순서가 바뀔수 있음. > Version을 체크하는 방법

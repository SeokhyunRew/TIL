<img width="425" height="319" alt="스크린샷 2025-08-23 오전 1 09 27" src="https://github.com/user-attachments/assets/fafa40be-46e1-419f-857f-0287761c731d" />

이 기법은 전적으로 운영체제가 관여를 합니다.

- Demand (요청)이 있으면 페이징을 하겠다.
- 페이징 기법을 사용한다고 가정하겠습니다.

Valid Invalid bit의 사용

<img width="414" height="337" alt="스크린샷 2025-08-23 오전 1 09 44" src="https://github.com/user-attachments/assets/c52225ec-0b52-4ae8-b7ba-9db8a65ce879" />

오른쪽은 swap Area(Backing Store)를 의미한다.

Invalid : 물리적 메모리에 없다! (Swap 되었어요)

- 최초 프로그램 실행시에는 전부 Invalid 상태입니다.
- Invalid인 경우 → MMU가 page fault를 발생시킴.
- 이건 IO작업이기 때문에 운영체제로 CPU가 넘어가게 된다.
- 
<img width="404" height="270" alt="스크린샷 2025-08-23 오전 1 10 28" src="https://github.com/user-attachments/assets/93dfc962-f93d-426d-b918-9755037a7bcc" />

- 이전에 배운 내용과 연결이 되는 부분이기도 하다.
  
<img width="407" height="337" alt="스크린샷 2025-08-23 오전 1 10 59" src="https://github.com/user-attachments/assets/9274f7ea-dbfd-497b-ad56-d9484123148e" />

- page fault → 운영체제 OS로 넘어감
  
<img width="409" height="254" alt="스크린샷 2025-08-23 오전 1 11 14" src="https://github.com/user-attachments/assets/9abd6918-20cc-48dc-b6d6-17b3e5dad034" />

Page Fault는 거의 0에 가까운 수치가 나오게 되기 때문에 효율적이다.

<img width="414" height="205" alt="스크린샷 2025-08-23 오전 1 18 13" src="https://github.com/user-attachments/assets/2b6ef6ea-bd87-4092-9ed7-e3e0330d5b2f" />

- page Replace Algorithm을 배워볼 것입니다. 
- 되도록이면 효율적으로 만들어서 page fault를 최소화 하는 방향으로 해야한다.

<img width="407" height="288" alt="스크린샷 2025-08-23 오전 1 19 01" src="https://github.com/user-attachments/assets/7154b3dc-96a1-426f-be32-5f9402697715" />

- 가장 좋은 것은 이건데 이론적이다. 미래를 알 수 없기 때문입니다. (미래를 안다는 가정하에 이것이 운영됩니다.)
- 가장 먼 미래에 참조되는 페이지를 교체합니다.
- 다른 알고리즘에 대한 upper bound를 제공합니다. 이것보다 좋은 것은 없으니 참고용으로 사용됩니다.

✅ 이제부터 실제 사용 알고리즘 → 미래를 알 수 없기 때문.

### FIFO

<img width="417" height="272" alt="스크린샷 2025-08-23 오전 1 19 28" src="https://github.com/user-attachments/assets/6545a656-1c7d-4a05-a48a-144985521192" />

- 특이한 점은 메모리 크기를 늘려주어도 페이지 fault가 더 많아지는 기이한 현상이 있고 FIFO Anomaly라고 부릅니다.
- 그래서 LRU를 좀 많이 사용하는 편.
  
<img width="413" height="226" alt="스크린샷 2025-08-23 오전 1 19 42" src="https://github.com/user-attachments/assets/c51968d5-de5e-45e4-adf9-b1d77e8193ca" />

- 가장 오래전에 사용된 것을 쫒아내겠다.
- 3번은 가장 오래전에 사용되었으니 그 자리에 5가 들어간 모습.
→ 미래를 모르니까 과거가 중요하고 그것을 기준으로 판단.

<img width="414" height="256" alt="스크린샷 2025-08-23 오전 1 20 03" src="https://github.com/user-attachments/assets/510647b3-693d-4ea2-b203-d3f13d206cd0" />

- 가장 적게 사용된 것을 쫒아낸다. (참조 횟수를 count)
- 동률일 경우, 마지막 참조시점을 한 번 더 판단하는 경우도 있다.
  
<img width="409" height="311" alt="스크린샷 2025-08-23 오전 1 20 38" src="https://github.com/user-attachments/assets/6ffbb0ab-ca7d-44dc-8263-3463f3ae1f47" />

- LRU는 과거에 많은 참조 횟수를 파악하지 못하지만, LFU는 가능하다.
- 하지만, 비록 4번은 1번이지만 막 4번 참조가 시작되는 시점이라면 문제가 될 수도 있다.

<img width="402" height="614" alt="스크린샷 2025-08-23 오전 1 21 41" src="https://github.com/user-attachments/assets/5b8fd245-c50f-4b9e-a134-fb59c5c2d9ac" />

- LRU는 구현이 매우 쉽다.
- list 하나를 만들고 제일 앞에 있는 것을 빼버리면 됨. 그리고 중간에 참조되면 순서 바꿔줌.

<img width="418" height="507" alt="스크린샷 2025-08-23 오전 1 22 03" src="https://github.com/user-attachments/assets/d978a1b8-6f5f-4099-8f02-a149469412d1" />

- LFU는 heap으로 합니다. 
- 뭘 뺴야하면 직계 자식이랑만 반복해서 비교한다.

<img width="414" height="349" alt="스크린샷 2025-08-23 오전 1 22 46" src="https://github.com/user-attachments/assets/e337b51d-4e4c-44e7-99b5-17090cd61553" />

> 저런 주소변환이 하드웨어적으로 일어나고 OS는 하는 것이 없다. 따라서 Paging System에서 LRU나 LFU는 할 수 없다..
> 
> 왜냐면 OS는 참조 횟수 이런걸 알수가 없다. 프로세스 자체가 OS가 관여안함.
>
> 사실 LRU LFU는 페이징 시스템에 맞는 알고리즘은 아니다.

<img width="412" height="302" alt="스크린샷 2025-08-23 오전 1 23 11" src="https://github.com/user-attachments/assets/8e8c6a20-efd3-46aa-8ad1-e09014e9fe9f" />

- 페이징 시스템에서는 Clock을 씁니다. (LRU 근사)
- Reference bit을 1로 세팅해서 페이지가 참조된 것을 표시 → 하드웨어가 수행합니다.
  
<img width="413" height="256" alt="스크린샷 2025-08-23 오전 1 23 26" src="https://github.com/user-attachments/assets/034bda61-320f-4c43-9d71-ed7249047996" />

Reference Bit이 1이면 최근에 참조가 되었으니 0을 찾아요. 근데 이건 최근에 참조된 것을 말합니다.

- 1이면 0으로 바꾸고 다음 것을 보는 것을 반복합니다.

> Reference Bit이 0이면 한바퀴 도는 동안에 페이지 참조 없었다는 뜻입니다.
>
> 어느 정도 참조 안된 것을 참조하기 때문에 LFU와 비슷.
>
> modified bit도 있다. 이게 0이면 그냥 쫒아내면 되는데 이게 1이면 수정된 사항을 반영을 해야합니다 !! (그냥 지우면 안됌)

- 지금 이런 알고리즘들은 프로레스로 묶어서 생각하는게 아니라 그냥 안쓰이면 내리는데.
- 프로그램이 잘 운영이 될려면 관련된 것들이 좀 잘 올라와 있어야 한다.
  
<img width="413" height="212" alt="스크린샷 2025-08-23 오전 1 24 01" src="https://github.com/user-attachments/assets/779a32e3-c13f-4808-87f7-935edaae670e" />

> 예를 들면, loop를 돌고 있다면..
>
> 그와 관련된 페이지를 다 준다면 loop도는 동안 페이지 fault가 나지 않겠죠.
>
> 그래서 프로그램마다 페이지를 적어도 몇개는 주어야 하는게 그런 개수가 존재한다는 것 입니다.
>
> 그래서 Allocation을 꼭 필요하다.
> 
> 이것을 **Allocation Scheme**라고 부른다.

<img width="416" height="206" alt="스크린샷 2025-08-23 오전 1 24 46" src="https://github.com/user-attachments/assets/a6c1da24-a5f5-45f9-b4b8-aa7b13b1393a" />

- 알고리즘을 쓰면 어느 정도 Allocation이 되는 효과가 있긴해요.
- ex) Working Set이나 PFF는 할당의 효과를 보장함.
- 위 : 다른 프로그램 페이지 쫒아낼 수 있음
- 아래 : 내 것만 관리

### Thrashing

<img width="409" height="565" alt="스크린샷 2025-08-23 오전 1 25 28" src="https://github.com/user-attachments/assets/521c55f1-a414-4df2-99a4-e043f10821b5" />

- 어떤 프로그램에게 최소한의 메모리를 할당하지 않을 경우 page fault가 자주 일어나는데 이런 경우를 쓰레싱이라고 한다.
- 프로그램 갯수가 늘어남에 따라서 CPU Utilization이 올라가다가 너무 많이 올라가면 내려가는데 이때 발생.
- 그래서 Allocation을 해주는 알고리즘들이 필요하다.

<img width="418" height="279" alt="스크린샷 2025-08-23 오전 1 26 05" src="https://github.com/user-attachments/assets/ab648241-eb76-43aa-a156-140d82808584" />

- Working Set 만큼 page를 못 얻으면 그냥 프로세스를 통째로 빼앗고 대기하게 된다.
  
<img width="411" height="246" alt="스크린샷 2025-08-23 오전 1 26 26" src="https://github.com/user-attachments/assets/1d0b2160-7f3e-440e-b0ca-21e6b63855ef" />

- Working Set은 미래를 보지 않는 이상 정확히 모르기 때문에 예측해야 하며 window를 설정한다.
- Working Set은 매번 바뀌는 모습.

<img width="410" height="339" alt="스크린샷 2025-08-23 오전 1 26 41" src="https://github.com/user-attachments/assets/97c64a49-53c1-45b6-9190-7d71212acb38" />

- 이건 page fault rate를 직접본다. page fault가 많이 일어나면 page 개수를 늘려준다. 프로그램마다 모니터링 하고 있음.
- 여기서도 메모리를 더 줄 수 없다? 그러면 프로그램을 swap out해버림.

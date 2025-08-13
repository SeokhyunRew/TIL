### IO Bound Job & Interactive Job

> CPU를 쓰는 단계와 IO를 실행하는 단계가 나누어지는데, 이때 XXX burst라고 부른다.
> 
> burst가 자주 바뀌는 작업이 있을 수 있고 그렇지 않은 작업이 있을 수 있다.

<img width="707" height="495" alt="스크린샷 2025-08-13 오전 5 43 53" src="https://github.com/user-attachments/assets/6ac461fd-d285-41bf-b210-668338378f99" />

- IO bound job은 빈도가 많은 대신 CPU적게 쓰고 CPU bound job은 진득하게 CPU오래 사용하는 작업이데
- 이때 IO bound job처럼 interactive한 job에 우선적으로 CPU를 할당할 필요가 있음.

### CPU Scheduler & Dispatcher

- 이건 하드웨어인가 소프트웨어인가? A: OS안에 있는 코드를 이렇게 부른다. 따라서, Software임.

CPU Scheduler

- Ready 상태의 프로세스 중에서 이번에 CPU를 줄 프로세스를 고름.

Dispatcher

- CPU 제어권을 CPU scheduler에 의해 선택된 프로세스에게 넘긴다.
- 이 과정을 context switch라고 한다.
- 준비 상태에 있는 프로세스가 CPU의 제어권을 넘겨받는 과정을 CPU dispatch
- 이때 걸리는 시간을 디스패치 지연시간 (dispatch latency)라고 하며 대부분의 문맥교환 오버헤드에 해당한다.

> 언제 스케줄링이 필요한가?

- Running → Blokced (IO 요청하는 시스템 콜)
- Running → Ready (할당 시간 만료)
- Blocked → Ready ( IO완료 후 인터럽트)
- Terminate

이때, 1과 4는 자진 반납이므로 비선점, 2와3은 선점형으로 볼 수 있다.
- 3번에서 원래는 CPU를 사용하던 프로세스에게 인터럽트를 통해서 잠시 뺏고 나서 다시 돌려주는 것이 일반적 이지만,그러지 않는 경우가 있다.
- IO프로세스에게 돌려주는 경우가 있는데 그것을 우선순위에 의해서 결정함.
- 앞에 인터럽트 당한 작업이 있더라도 경우에 따라서 바로 돌려줌.

>자진 반납 : nonpreemptive
>
>강제로 뻇음 : preemptive

### CPU Scheduling Issues

1. 누구에게 당장 CPU를 줄 것인가?
2. CPU를 일단 줬으면 이 프로그램에게 CPU를 계속 주느냐 아니면 중간에 제어권을 가져오는가?

- non preemptive (비 선점형) : 한 번 제어권을 넘기면 강제로 받을 수 없다.
- preemptive(선점형) : 제어권을 다시 가져온다 (주로 이것을 사용)

### Scheduling Criteria (성능 척도)

### 시스템 입장에서 성능 척도 → CPU하나 가지고 일을 최대한 많이 시키면 좋다

- CPU utilization(이용률) → (CPU가 일한 시간 / 전체시간)이 비율이 높을수록 좋다
- Throughput (처리량) → 주어진 시간 동안 몇개의  작업을 완료했는가??

### 프로그램 입장에서 성능 척도 → 내가 CPU를 빨리 얻어서 빨리 끝나면 좋다
- Turnaround time (소요시간 + 반환시간) → 쓰러 들어와서 다 쓰고 나갈때까지 걸린 시간을 의미한다.
- waiting time (대기 시간) → CPU를 쓰기 위해서 ready queue에서 기다린 시간.
- Response time( 응답 시간) → 처음으로 CPU를 얻기까지 걸린 시간

> 특히 응답시간을 둔 이유는??
>
> 처음으로 응답을 얻는 시간이 사용자 사용성과 관련이 깊다.

### FCFS (First-Come First-Served)

먼저 온 순서대로 처리해주는 스케줄링.
- 비선점형 스케줄링이며 Interactive Job에 불리하다.
- 만약 제일 오래 걸리는 프로세스가 먼저들어오면 waiting time이 길어진다
- 그런데 반대면 waiting time이 아주 짧다는 특징이 있다.

> FCFS는 결국 처음 도착한 사람에 따라서 waiting time이 다양하게 달라짐.
>
> 오래 걸리는 게 먼저오면 **convoy effect**라고 부릅니다. 

### SJF (Short-Job first)

CPU를 짧게 쓰는 작업들에게 CPU를 주는 스케줄링.
- 전체적 행복한 결과가 나오게 된긴 한다.
- Queue가 짧아지고 average waiting time이 가장 짧아지는 스케줄링 방식.
- preemptive하게 적용하면 더 짧은 프로세스가 오면 뺏어서 사용하게 함. 이게 average time이 optimal하게 짧음
- 이럴경우 SRJF라고도 부른다.
- non preemptive는 끝나는 시점에 CPU스케줄링을 하지만, preemptive는 새로운 작업이 들어오면 바로 CPU 스케줄링을 함.

> 문제점 1.  Starvation
> 
> 극단적으로 CPU시간이 짧은 것을 선호 따라서 CPU 사용 시간이 긴 프로세스들은 영원히 사용하지 못할수도 있음.

> 문제점 2. 
>
> CPU사용시간을 미리 알 수 없다는 문제점이 있다.

본인이 얼마나 쓰고 나갈지 조차 알 수가 없으니 실제로는 SJF를 쓰기는 어렵다. 
다만 exponential averaging을 이용해서 추정은 할 수 있는데, 이는 프로그램이 CPU를 얼마나 쓸지 과거의 흔적에서 유추한다.

### Priority Scheduling

우선 순위 스케줄링 우선 순위 높은 것 먼저주는 스케줄링.
- preemptive 와 non-preemptive 둘다 있음 이것 역시.
- 각 프로세스 마다 정수로 Priority number를 받고 일반적으로는 작은 숫자가 우선순위가 높다는 의미이다.
- SFJ는 CPU시간이 적은게 우선 순위라고 볼 수 있으며, SFJ는 일종의 priority scheduling 

> 이것 역시 starvation 문제가 발생 가능하다.
>
> 이에 따라서, Aging 기법을 도입할 수 있다. → 아무리 우선 순위가 낮더라도 오래 기다리게 되면 우선 순위가 높아지게 된다. → starvation을 막자.

### Round Robin → 현대적 스케줄링

할당 시간을 설정해서 나누어주는 선점형 스케줄링이며 지금까지 강조해왔던 스케줄링이다.

- 동일한 크기의 할당 시간을 나누어주며, 응답시간이 빨라진다는 장점이 있다.
- 누가 CPU를 오래 쓸지 모르는 상황에서 굳이 예측할 필요없이 short job은 빠르게 나가게 된다.
- **기다리는 시간은 CPU사용 시간에 비례하게 된다는 특징**이 있음.
- SJF보다 turn around타임이나 waiting 타임은 길어질 수 있지만 최초 CPU를 얻는 Response time은 짧아집니다.

> q: time unit 
>
> n: 프로세스 개수
>
> 어떤 프로세스도 (n-1)*q를 기다리면 최초 응답을 받을 수 있다.
>
> q를 너무 크게 잡으면 FCFS와 같아지고, q가 너무 작아지면 컨텍스트 스위칭 비용이 너무 많이 발생한다.
>
> q는 적당해야 좋고 10~100 ms이다 로 알려져 있습니다.

### Multilevel Queue

줄을 여러개로 만들고 줄마다 우선 순위가 다른 케이스이다.
- 우선 순위를 극복 못함
- Ready Queue를 여러개로 분할
- foreground (interactive) : RR을 쓰는 것이 좋다.
- background (batch - 인간과 소통 상호작용 x) : FCFS를 사용하는 것이 좋다.

> 위처럼 큐에 따른 스케줄링 방법이 다르게 존재한다.
> 
> 이에 따라서, 큐에 대한 스케줄링이 필요

- Fixed priority scheduling 줄에 대한 우선 순위가 바뀌지 않음 → starvation 발생가능.
먼저 해야하는 줄을 다하고 나중 줄을 처리한다는 말임.
- Time slice 각 큐에 CPU time을 적절한 비율로 할당 RR에 0.8 FCFS에 0.2 정도.

### Multilevel Feedback Queue

우선 순위가 바뀔 수 있는(줄이 바뀔 수 있는) 스케줄링 방식이다.
- 승격 강등 기준을 정해 주어야 한다.
- 보통은 처음 들어오는 프로세스는 우선순위 높게 하고 여기에 RR을 함 
- 밑으로 갈수록 q를 길게하고 맨 아래 큐는 FCFS사용함.
- 각 줄에서 할당 시간내에 처리를 못하면 아래로 계속 강등 됌.

> 라운드 로빈 만으로도 힘들기 때문에 이런 방식을 사용할 수 있다.

### Multiprocessor Scheduling(CPU 다수인 특수 스케줄링 방식)
- 만약 어떤 job은 특정 CPU가 해야하는 경우. ex) 꼭 원하는 헤어 디자이너에게 서비스 받기.
- 이렇게 CPU가 많다면 load sharing이 필요하다, 특정 CPU만 일을 몰아서 하면 안되기 때문이다.
- 별개의 큐를 둘수도 있고 공동 큐를 사용할 수도 있습니다.

> (대칭형 다중 처리)Symmetric Multiprocessing (SMP)
- Symmetric이란 대등하다는 뜻이다.
- CPU가 알아서 스케줄링을 처리한다.

> (비대칭형 다중 처리)Asymmetric Multiprocessing
- CPU가 여러개 있는데 하나의 CPU가 전체적인 컨트롤을 담당하고 나머지 CPU는 그것에 따른다.

### Realtime Scheduling

- Hard real-time system

> 이건 반드시 시간안에 끝나야 하는 작업을 의미하며, 보통 미리 스케줄링 하고 적재적소에 배치 되도록 한다.
> 
> 또는 데드라인이 얼마 남지 않은 것들을 먼저 처리하는  EDF(Earliest Deadline first)를 사용한다.

- Soft real-time system

> 일반 프로세스에 비해 높은 priority를 갖도록 해야하며, 반드시 시간 안에 끝날 필요는 없다.

### Thread Scheduling

- Local Scheduling

> User Level thread의 경우 사용자 수준의 thread library에 의해 어떤 쓰레드를 스케줄할지 결정
>
> 프로세스에게 CPU가 갔을 때 프로세스가 분배 OS가 하는 것이 아니다.

- Global Scheduling

> 커널 레벨 쓰레드의 경우 일반 프로세스와 마찬 가지로 커널의 단기 스케줄러가 어떤 thread에게 작업을 줄 지 결정한다.

### 알고리즘 평가 Algorithm Evaluation

> 1. Queuing Models
> 
> 굉장히 이론적인 모델. 확률 분포로 주어지는 arrive rate과 service rate를 통해서 평가
>
> 각종 performance index 값을 계산에 의해서 산출

> 2. Implementation (구현) & Measurement(성능 측정)
>
>1과 반대인. 실제 시스템에 알고리즘을 구현하여 실제 작업에 대해서 성능을 측정한다.

> 3. Simulation (모의 실험)
> 
> 시뮬레이션 프로그램이 돌려준다.
> 
> 이때 예제 단 하나를 두고 알고리즘이 좋다고 주장하면 적당하지 않다. 그래서 보통 실제 시스템에서 CPU burst 패턴을 가져와서 실험을 한다. 이런 데이터를 보통 Trace라고 부름.

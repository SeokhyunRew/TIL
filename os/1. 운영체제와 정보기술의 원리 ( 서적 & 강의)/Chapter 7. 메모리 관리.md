### Logical Address (virtual address)

- 프로세스가 시작되면 0번지 부터 시작하는 가상의 독립적 공간을 뜻한다.

### Physical Address

- 메모리에 실제로 올라가는 위치.
- 하단 부분에는 커널이 있고 위에 프로세스가 올라가 있다.

### 주소 바인딩 : 주소를 결정하는 것

Symbolic Address → Logical Address → Physical Address

- Symbolic Address : 프로그래머들이 프로그램을 만들 때 숫자를 가지고 코딩하지 않는다 이런 코드 그대로를 Symbolic Address라고 부른다. 

프로그래머 입장에서는 Symbol로 된 주소를 사용한다는 의미.
- 논리적 주소가 물리적 주소로 언제 결정이 되는가? 방법이 3가지이다.

### Complie time binding

- 컴파일 시 알려짐.
- 아래 사진에서 컴파일 시점에 Logical이 아니라 물리적 주소를 가짐.

물리적 메모리 주소가 많이 비어 있음에도 0번지 부터 미리 올려야 하고 컴파일 시점에 결정이 되어서 매우 비효율적.

Logical Address지만 물리적 주소로 그대로 바인딩 되기 때문에 절대 코드(Absolute code)라고 부른다.
그래서 이건 메모리 위치를 바꾸고 싶으면 다시 컴파일 해야함.

### Load time binding

- 실행이 시작될 때 바인딩

ex) 500번지 부터 비어 있었다 그래서 올렸음.
어느 위치든 올라갈 수 있으니 재배치가능 코드 (relocatable code)라고 부름.

### Execution time binding(Run time binding)

- 실행하다가 메모리 상 위치를 바꿀수도 있음.

> 실행시에 결정된다는 것은 Load time과 같으나 실행 도중 바뀔 수 있다는 점이 있다.
>
> CPU가 주소를 참조할 때마다 binding을 점검하고 하드웨어적인 지원이 필요하다 (MMU)

<img width="607" height="408" alt="스크린샷 2025-08-23 오전 12 17 18" src="https://github.com/user-attachments/assets/8c0827a8-549e-44f2-9b7e-7109c154feff" />

물리적 메모리가 결정되는 것이 주소 바인딩이라고 부름.
- **CPU가 바라보는 주소는 물리적 주소일 것 같지만 사실 Logical Address임**
- 실행파일이 올라가는 위치는 메모리에 500 ~ 530이 올라가지만 컴파일 코드로 되어 있는 위치는 logical 위치로 남아있을 수 밖에 없어요.
- CPU는 매번 Logical Address를 바라보고 전달시에 변환해서 주어야 하는 것입니다.

### Memory-Management Unit (MMU)

- 주소 변환을 위한 하드웨어 : Logical Address를 Physical Address로 매핑해주는 하드웨어
  
    <img width="378" height="266" alt="스크린샷 2025-08-23 오전 12 20 55" src="https://github.com/user-attachments/assets/3c271727-34c6-41d9-a951-82616d978037" />
    
- 레지스터 두개를 통해서 주소변환을 하게 됩니다.

> User 프로그램도 logical address만 바라보고 CPU도 그렇다.
> 
> 물리적 주소로는 MMU가 변환해줄 뿐이다.

<img width="380" height="288" alt="스크린샷 2025-08-23 오전 12 21 31" src="https://github.com/user-attachments/assets/35101a64-e876-4a8e-93db-83cddef6429a" />

- ex) CPU는 요청합니다 346번 위치에 있는 값을 달라. 이건 **가상 주소**입니다.

> relocation register( base register)와 limit register 두개를 통해서 주소 변환을 하게 됩니다.
> 가상 메모리는 0번지 부터 하니까 0부터 346 떨어진 값을 달라고 한 것이고.
>
> 만약 이 프로세스에 대한 값이 14000 부터 시작한다면, 14346이 맞다.
> 물리적 메모리 시작 위치에서 논리 주소를 더해주면 됩니다.

- 또한, limit register를 사용하는 이유는 저걸 넘으면 안되기 때문입니다. (프로그램의 크기를 담고 있다.)
- 만약 악의적인 프로그램이어서 3000이상을 요청할 수 있기 때문임.
  
<img width="412" height="303" alt="스크린샷 2025-08-23 오전 12 23 18" src="https://github.com/user-attachments/assets/04fdbb5e-cea2-4510-8cfa-d4766915cee5" />

- 그래서 limit register를 넘어가는 요청을 하면 trap이 걸리게 되고 왜 그게 발생했는지 확인하게 됩니다.

<img width="583" height="374" alt="스크린샷 2025-08-23 오전 12 23 58" src="https://github.com/user-attachments/assets/dee43bfb-6b89-4169-b270-fb437b267673" />

> 좋은 프로그램일 수록 자주 사용하는 부분이 한정적이다.
> 
>오류 처리 루틴이 많기 때문에 통째로 올리는 것은 비효율적이기 때문에 필요한 상황이 생기면 그때 메모리에 올리는 것을 Dynamic Loading이라고 부른다.

> 현재 시스템도 필요할 때 올리고 내리는데 그건 OS가 지원해 주는 Paging 기법이고
> 
> 보통 다이나믹 로딩은 프로그래머가 직접 하도록 만든 개념이다.

<img width="406" height="209" alt="스크린샷 2025-08-23 오전 12 25 04" src="https://github.com/user-attachments/assets/707d5eb4-2355-45bb-a5e2-72b6cd575cd7" />

- 이건 다이나믹 로딩과 구분을 해야한다.
- 역사적으로 조금 다릅니다. 초창기 컴터 시스템은 메모리가 너무 작아서 프로그램 하나를 올리는 것도 불가능했다.
- 그래서 프로그래머가 쪼개서 메모리에 올려놨음. 수작업으로 코딩함. 이것을 Manual Overlay라고 부릅니다.

> 다이나믹 로딩도 결과적으로는 그렇게 되지만, 라이브러리를 통해서 되는 차이가 있다.

<img width="412" height="277" alt="스크린샷 2025-08-23 오전 12 25 40" src="https://github.com/user-attachments/assets/931038cc-6a38-4210-a3c9-0a323353a1f1" />

- 통째로 Swap Area로 쫒아내는 것.

> 그 공간을 Backing Store이라고 부른다.
> 
> 이런 것을 중기 스케줄러가 보통 담당함.

- Swaping 시스템은 앞의 바인딩과 연결해서 생각할 필요가 있다.
- Compile time binding이나 Load time binding을 사용한다면 쫒아냈다 돌아올 경우 원래 위치로 와야한다. 그래서 스와핑의 효과를 극대화 하기는 힘듬.
- 그래서 Swapping을 효과적으로 하기 위해서는 Runtime binding이 지원이 되는 것이 좋겠죠.

> 뒤에서 다시 배우겠지만, 
> 대부분 디스크 헤드가 움직이는 seek time이 대부분 시간을 참여하는데 
> 스왑 같은 경우는 메모리가 커서 transfer time도 고려해야 한다.
>
> 요즘에 와서는 page가 스왑 아웃되도 스왑 아웃되었다는 표현을 쓰지만, 
> 원래 대로라면 프로그램 전체가 스왑 아웃 되어야 한다.

<img width="408" height="289" alt="스크린샷 2025-08-23 오전 12 27 30" src="https://github.com/user-attachments/assets/9cd09676-d9ab-4744-a2dc-c1a129b96e54" />

- 프로그램을 작성하고 컴파일하고 Link해서 프로그램을 실행하게 됨.

> 링크는 여러 군대에 존재하던 파일을 link해서 합치는 개념임.
> 
> 함수를 불러와서 쓰는 library도 결국 링킹임.

- Static linking → 라이브러리 사용
- Dynamic linking → 라이브러리 함수가 내 코드에 컴파일 안되어 있는대로 있다가 사용시 링크.
- 실행 파일에는 라이브러리가 별도의 파일로 존재하고 stub라는 라이브러리를 찾을 수 있는 포인터만 포함시켜 놓는 것임.

> Dynamic linking을 해주는 라이브러리를 shared library라고함.

<img width="411" height="295" alt="스크린샷 2025-08-23 오전 12 28 43" src="https://github.com/user-attachments/assets/bcb4b3e2-5521-4ae0-822d-df7b06327c31" />

- 이제 물리적인 메모리를 어떻게 관리할 것인가??를 배워야 한다.
- 사용자 프로세스 영역의 할당 방법은 연속 할당과 불연속 할당이 있다.

연속할당
- 프로그램이 한 군대에 통째로 올라간다.

불연속 할당
- 프로그램을 구성하는 주소 공간을 잘개 쪼개서 올린다.
  
### 고정 분할 방식

- 미리 파티션을 나누어 놓는 방식.

외부 조각
- 이 프로그램을 올리고 싶은데 메모리가 더 작아서 사용하지 못하는 메모리인 경우.

내부 조각
- 프로그램의 크기가 분할보다 작아서 내부에서 낭비되는 메모리 조각.

두 조각 모두 발생 가능하다.

### 가변 분할 방식

- 내부 조각은 없지만, 외부 조각이 발생할 수 있다.

<img width="410" height="292" alt="스크린샷 2025-08-23 오전 12 31 00" src="https://github.com/user-attachments/assets/32651477-fd9e-4d7d-943f-44766e02c133" />

- 또한, Hole들이 여기저기 산발적으로 생길 수 있다.
- 이럴 때  OS는 hole들을 관리하고 어떤 hole에 프로그램을 올려야 하는가 판단해야함.

<img width="378" height="263" alt="스크린샷 2025-08-23 오전 12 31 26" src="https://github.com/user-attachments/assets/77373b43-ce33-4307-8b8c-ad4d56ac98a6" />
    
> 그것을 Dynamic Storage-Allocation Problem이라고 부릅니다.
> First-fit 이나 Best-fit이 당연히 Worst보다는 좋음
> First는 공간을 찾는 오버헤드가 적으니 Best에 비해서 시간적 장점은 가지고 있음.

<img width="412" height="180" alt="스크린샷 2025-08-23 오전 12 32 14" src="https://github.com/user-attachments/assets/ce52df54-ace6-4211-aabb-030043e25ee3" />

- Compaction이란? hole들을 밀어가지고 하나의 큰 구멍을 만드는 작업. 그렇게 호락 호락한 방법은 아닌편입니다.
- 전체를 미는 것보다는 작은 이동을 통해서 공간을 만드는 것이 효율적일 수 있다 (다른 조그만한 hole들이 존재해도)

### Paging

<img width="330" height="504" alt="스크린샷 2025-08-23 오전 12 36 45" src="https://github.com/user-attachments/assets/d33cf3bb-9abc-4748-a157-9e9a38efefe5" />

- 동일한 페이지 단위로 어디든지 올릴 수 있다, 이전과 다르게 페이징 테이블이 필요함(좀 더 복잡한 주소 기록 로직)

> 페이지 테이블 → 각각의 논리적 페이지가 논리적 메모리 어디에 올라가 있는가?
> 
> 페이지가 들어갈 수 있는 공간이 촤즉에 페이지 프레임이라고함 (물리적)
> 
> 페이지 테이블은 페이지 개수만큼 **엔트리**가 필요함.

<img width="335" height="244" alt="스크린샷 2025-08-23 오전 12 38 00" src="https://github.com/user-attachments/assets/d34a2578-64c5-4493-b1e9-6894c4ef5c09" />

- P: 페이지 번호 , d : 얼만큼 떨어진지 Offset
- f : 몇번째 프레임인지로 바로 변환할 수 있다.

> 결국 페이지 위치만 메모리 어디 올라가는게 바뀌는거고 Offset은 바뀌지 않는다.

### 페이징 테이블은 어디에 존재할까?테이블은 어디에 들어가 있을까요? 

- 이전에는 레지스터를 썻으니까 이건 CPU안에 있었다.
- 페이지 크기는 보통 4KB라 주소 공간이 매우 크고, 프로그램 하나당 엔트리 100만개 정도 필요한데 이건 CPU에 넣을수는 없다. 
- 그리고 또 프로그램마다 페이지 테이블이 별도로 존재하기 때문에 캐시에 넣기도 힘들다.

**따라서 이건 메모리에 들어가 있습니다.**

> 원래는 메모리에 접근하기 위해서는 페이지 테이블에 접근 해야하는데, 이게 메모리에 있으니
>
> 메모리 접근을 위해서는 2번의 메모리 접근이 필요하다고 생각하면 된다.

<img width="378" height="156" alt="스크린샷 2025-08-23 오전 12 41 22" src="https://github.com/user-attachments/assets/28c826c4-1e8c-47ed-8758-12e5f964ff54" />

- 두 개의 레지스터도 바뀌게 되었다.

이걸 좀 효율적으로 하기 위해서 별도의 하드웨어가 필요하다.
- **assosiatice register** 혹은 **translation look-aside buffer TLB (일종의 캐시)**
- 메인 메모리 보다는 빠르고 CPU보다는 느린 그 사이의 계층.
    
 <img width="387" height="287" alt="스크린샷 2025-08-23 오전 12 43 28" src="https://github.com/user-attachments/assets/24cafbec-0a16-4bb0-8fa3-e31035bdeb2e" />

- TLB는 데이터를 보관하는 캐시 메모리보다 주소 변환을 위한 캐시 메모리. 빈번하게 참조되는 엔트리를 보관하고 있다.
- 근데 TLB에는 몇번째 엔트리 번쨰인지 알 수가 없으니까?? TLB는 framenumber도 가지고 있어야한다.
- TLB는 반드시 전체를 search 해야하기 때문에 병렬 탐색이 가능한 Associative Register으로 구현이 되어 있음.
- 근데 page table은 full scan이 필요없어 왜냐면 배열이 p번째에 p가 있다.
- TLB도 context switch마다 flush필요 (프로그램마다 다르니까)

<img width="501" height="306" alt="스크린샷 2025-08-23 오전 12 45 49" src="https://github.com/user-attachments/assets/18b6edd6-f475-483d-9b43-d7157f43f7c4" />

- TLB 접근 시간을 입실론이라고 하면, 이는 메인 메모리 접근 시간 1보다 일반적으로 작다.
- TLB로 부터 주소가 찾아지는 비율이 알파라면,  

> Hit : 메모리 접근 시간 + 찾는 시간
> 
> Miss :  페이지 테이블 접근 + 메모리 접근 + 찾는 시간

- 페이지 테이블만 있을 때는 2니까 훨신 작은 시간이 걸린다.

✅ 2단계 페이지 테이블

<img width="463" height="730" alt="스크린샷 2025-08-23 오전 12 46 42" src="https://github.com/user-attachments/assets/69621b04-9057-41b9-baf6-ad9ccaa72be6" />

> 안쪽 페이지 테이블과 바깥쪽 페이지 테이블이 있는 구조, 왜 2단계 테이블이 필요한가?

32bit를 쓰니까 이걸 기준으로 설명하면 메모리 주소는 (물리적) 2^32-1까지 주소 표현이 가능하다.

- 2^10 : K
- 2^20 : M
- 2^30 : G

> 32비트면 4G를 표시할 수 있다. 이걸 페이지 단위로 쪼갭니다.
> 
> 근데 한 페이지는 4KB입니다. 따라서, 1M개의 페이지가 나옵니다 (2^20)

- 그런데, 페이지 테이블도 메모리에 들어가야하고 프로그램마다 페이지 테이블이 다르니 전부 메모리에 넣으면 공간낭비가 심하다. 
- 엔트리 하나 크기가 4B인데 1M개 있으니 한 프로그램당 4MB의 페이지 테이블이 프로세스마다 필요하다.

이를 2단계 페이지 테이블을 통해 절약이 가능하다.

<img width="383" height="367" alt="스크린샷 2025-08-23 오전 12 48 52" src="https://github.com/user-attachments/assets/dd771a32-fcfa-46d7-9876-2ac60d9f1812" />

- p1과 p2로 주소 체계를 두개로 분류화

<img width="377" height="180" alt="스크린샷 2025-08-23 오전 12 49 09" src="https://github.com/user-attachments/assets/0d372de1-53e9-436f-929a-4d6316712759" />

- 바깥 페이지 테이블에서 p1을 넣고 다음 정보를 얻는다. "안쪽테이블 중에 어떤 페이지 테이블인가?"
- (바깥 페이지 테이블 엔트리 하나당 안쪽 테이블이 하나씩 만들어짐)

p2: 안쪽테이블의 p번째…

> 여기서 기억해야할 것은 안쪽 페이지 테이블 크기가 페이지랑 똑같다는 점이다.
> 
> 안쪽 페이지 테이블은 페이지화 되서 메모리에 들어가있음 (4KB).
> 
> 그리고 엔트리 크기가 4B니까 페이지 테이블 하나당 1K개의 엔트리.


⭐ Bit수 알기

페이지 하나의 크기는 4KB (2^12Bit)

<img width="386" height="222" alt="스크린샷 2025-08-23 오전 12 50 37" src="https://github.com/user-attachments/assets/1af09620-6801-4da8-8d02-00b1dfc93319" />

2^12 Bit를 구분하기 위해서는 12Bit가 필요하다. 
Q: p1 & p2는 20Bit 어떻게 나눌 수 있는가?

> 안쪽 페이지 테이블은 페이지화 되어서 들어가는 4KB 엔트리는 1K개 하나당 4B니까
>
> 안쪽 테이블에 엔트리가 1K개가 있고 1K개의 엔트리를 구분하기 위해 p2에는 10Bit가 필요하다.
> 
>전체 32에서 22빼면  바깥을 위한 페이지 테이블은 10Bit를 할당한다.

- 시간은 더걸리지만 페이지 테이블 공간을 줄일 수 있다.
- 페이지 테이블의 엔트리는 사용되지 않는다고 해도 logical memory의 크기만큼 만들어져야 하는데 2단계 테이블을 쓰면 그걸 해소하면 가능. 
- 바깥 테이블은 메모리 크기만큼 만들어지지만 내부 테이블은 그냥 안만들어져 있다.
  
<img width="473" height="241" alt="스크린샷 2025-08-23 오전 12 52 46" src="https://github.com/user-attachments/assets/91cfaf26-eb0b-4ec5-8811-42cd66583881" />

<img width="414" height="216" alt="스크린샷 2025-08-23 오전 12 53 08" src="https://github.com/user-attachments/assets/750655b4-27f9-4e63-9c53-b0bb69e41561" />

다단게 페이지 테이블을 사용하면 n단계 수 만큼 n번 접근해야 하지만, TLB를 사용하면 많이 효율화 할 수 있음.

<img width="409" height="370" alt="스크린샷 2025-08-23 오전 12 53 38" src="https://github.com/user-attachments/assets/2a77b8ea-d380-4497-afac-7b407719b51c" />

Logical 테이블에 있는 것 만큼 엔트리가 존재하고 page frame 번호가 있다.

- 그런데 사실 페이지 테이블에 부가적인 bit이 저장됩니다.
- 페이지를 쓰던 말던 엔트리가 존재하긴 함. 왜냐면 엔트리는 맥시멈 페이지 개수만큼은 생겨야 하기 때문이다.

> 그래서 안쓰는 페이지에 대해서 invalid가 쓰여있다. valid라는 것은 실제로 메모리에 올라와 있다는 뜻입니다.

<img width="416" height="284" alt="스크린샷 2025-08-23 오전 12 55 09" src="https://github.com/user-attachments/assets/548a0f64-aa7f-4d20-b26f-e8588a12bb02" />

- swap 되어 있다.
- 사용하지 않는다.

<img width="406" height="289" alt="스크린샷 2025-08-23 오전 12 56 13" src="https://github.com/user-attachments/assets/5c4d36d5-4d58-465c-804a-c504091dab47" />

- 기존에 페이지 테이블의 문제는 너무 큰 용량을 차지한다는 문제였습니다. 이걸 해결하고자 역페이지 테이블이 나옴.

<img width="417" height="326" alt="스크린샷 2025-08-23 오전 12 56 44" src="https://github.com/user-attachments/assets/48aa3302-d39e-4300-867d-4c378792b4e4" />

> 원래는 프로세스마다 테이블이 존재했다.
> 
> 그런데 시스템 안에 page 테이블이 하나 존재하고,
> 
> 프로세스의 페이지개수만큼 존재하는 것이 아니라 물리적 페모리의 페이지 수 만큼 존재한다.

- 페이지의 개수만큼 엔트리 존재
- 페이지 프레임의 f번째 엔트리를 가면 논리적 페이지 정보가 나오도록 되어 있다.
- 원래는 logical address를 physical을 바꾸는 것이다. 그런데 위의 내용은 반대임. 그래서 엔트리를 다 찾아봐야함.
- 그리고 도대체 페이지 번호 p가 어떤 프로세스인지도 저장을 해야하기 때문에 pid를 저장해야함.
- 공간을 줄이겠다고 너무 탐색이 오래 걸리기 떄문에 연관 레지스터 써서 병렬 검색하는 것이 좋다.

<img width="402" height="231" alt="스크린샷 2025-08-23 오전 12 57 59" src="https://github.com/user-attachments/assets/8b96acb0-b7d9-4605-9e7f-92047d645d10" />

그리고 페이지들 중에는 공유할 수 있는 페이지들이 존재합니다.

<img width="415" height="361" alt="스크린샷 2025-08-23 오전 12 59 02" src="https://github.com/user-attachments/assets/c8ddce3f-9594-42d9-9c3c-281634ef59a8" />


> 만약 같은 코드를 가지고 수행을 한다면. 공유하는 코드는 공유해도 괜찮다.
>
> 이런 코드들은 Read Only로 해야하고 동일한 logical address에 위치해야한다.
>
> 한 copy만 올려놓으면서 사용할 수 있다.

> private code나 data는 각각의 다른 프레임으로 매핑되도록 해야할 것.
>
> "왜 동일한 logical address라는 조건이 필요한가요??"
> 
> 주소 바인딩에서 주소들이 적혀있다.
> 
> 코드 안에서 logical address가 적혀있다.
> 
> 코드는 바꿀 수 없기 때문에 이것들이 동일해야함.

<img width="414" height="277" alt="스크린샷 2025-08-23 오전 1 00 30" src="https://github.com/user-attachments/assets/a5e4e6a5-efc3-4d42-8d4b-27f0e29f55ed" />

페이징 기법은 프로그램을 같은 페이지 단위로 쪼개는것 세그멘테이션은 의미 단위 (code,data,stack)으로 쪼갠 것.

<img width="410" height="282" alt="스크린샷 2025-08-23 오전 1 00 46" src="https://github.com/user-attachments/assets/56f66381-830f-43ef-b559-84de7f551826" />

주소 변환이 paging과 비슷하다.


<img width="416" height="343" alt="스크린샷 2025-08-23 오전 1 01 01" src="https://github.com/user-attachments/assets/e59e47f3-593e-46fd-98e5-eace68d09b87" />

> CPU에서 준 segment 주소를 두 구역으로 나눈 segment 번호와 offset
>
> 엔트리 개수도 정해져 있지 않음. 세그먼트 개수에 따라서 엔트리 개수가 만들어지게 된다.

- 엔트리에 limit이라는 정보도 알고 있어야 함. 
- paging은 page 길이가 같았지만 세그먼트는 길이가 다르기 때문에 페이지 테이블에 limit줘야함.
- 그리고 페이징은 frame 번호로 기록이 되어 있었는데, 그건 페이지 크기가 같았을 때 가능했던 이야기고 세그먼트는 같지 않기 떄문에 시작 위치를 바이트로 주어야 하고. 
- 중간 중간 구멍 hole문제는 세그먼트에서 여전히 처리해야할 문제

<img width="416" height="294" alt="스크린샷 2025-08-23 오전 1 02 06" src="https://github.com/user-attachments/assets/ca735f9d-4e50-4a84-900f-33190e84a530" />

> 그럼 장점은 ?

- 의미 단위로 쪼개는 것이기 때문에 의미 단위로 수행하는 일은 세그멘테이션은 유리하다.
- 특히 Protection을 하게 되면 의미 단위로 의미 부여를 할텐데 그럴 때 유리하다고 합니다.
- 그리고 sharing을 할때에도 의미 단위로 하기 때문에 paging보다 Segment가 유리합니다.

<img width="420" height="293" alt="스크린샷 2025-08-23 오전 1 02 44" src="https://github.com/user-attachments/assets/f4695f37-599a-4f8b-a277-93ceee29792f" />

- 세그먼트는 갯수가 페이지 만큼 많지는 않아서 효율적이다.
- 테이블로 인한 메모리 낭비로 따지자면 페이징
  
<img width="412" height="339" alt="스크린샷 2025-08-23 오전 1 03 15" src="https://github.com/user-attachments/assets/210673d1-1873-4458-89c7-ffe38eeade2b" />

세그먼트가 5개인 예시 저렇게 주제별로 만들어져 있음.

<img width="410" height="428" alt="스크린샷 2025-08-23 오전 1 03 29" src="https://github.com/user-attachments/assets/da96cd49-070c-41a8-9af2-ec0e7827bd98" />

세그먼트는 공유가 유리하다고 했는데, 공유하는 예시는 다음과 같다. 0번을 공유하는 예시.

✅ 세그멘테이션 + 페이징

<img width="408" height="353" alt="스크린샷 2025-08-23 오전 1 03 41" src="https://github.com/user-attachments/assets/6d26a4ac-a87d-4af6-aa0d-1a87e882f4ee" />

- 먼저 세그먼트에 따라서 변환을 한다. (세그먼트 테이블을 탐색)
- 세그먼트 하나가 여러개의 페이지로 구성이 되어 있어서 메모리에 올라갈 때에는 페이지 단위로 올라간다고 합니다.

> allocation 문제가 발생하지 않아요 
>
> 의미 단위로 하는 것은 세그먼트 레벨에서 하는겁니다.

- 거의 세그먼트만 쓰는 프로그램은 없음 세그먼트를 이렇게 섞어서 쓰는 경우가 많다.
- 2단계로 해야하는데 세그먼트당 페이지 테이블이 존재하게 됨. 
- Q : 페이지 테이블에 엔트리가 몇개? 세그먼트 길이를 보면 알 수 있다.
- d가 나중에 페이지 번호와 offset으로 다시 나눠진다는 것을 숙지하자.

### 💡 결국 주소 변환에서 운영체제의 역할은?

> MMU라는 하드웨어가 이것을 해주어야함 운영체제의 역할은 없다. **단 하나도**. 
>
> 근데 왜 ? 이걸 하드웨어가 해주어야 할까요?

→ CPU clock 사이클 마다 주소 변환을 해야하면 CPU가 운영체제로 너무 많이 넘어갔다 왔다 갔다 하기 때문에 하드웨어의 일이 맞다.

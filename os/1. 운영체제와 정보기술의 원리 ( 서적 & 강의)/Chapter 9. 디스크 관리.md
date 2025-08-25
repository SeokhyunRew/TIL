<img width="702" height="465" alt="스크린샷 2025-08-24 오전 1 48 03" src="https://github.com/user-attachments/assets/33e4679d-04cd-4f62-9790-3d5af103f1d7" />
<img width="798" height="748" alt="스크린샷 2025-08-24 오전 1 49 40" src="https://github.com/user-attachments/assets/bbaa7164-6c2b-433d-92e0-07aad4604169" />
<img width="709" height="732" alt="스크린샷 2025-08-24 오전 1 50 08" src="https://github.com/user-attachments/assets/e5244156-93f5-40f9-a711-c5cfb74a12bb" />

> 예전에는 사용자가 직접 포메팅을 해야했는데.. 지금은 포메팅이 되어있음.

- 각 섹터는 header + trailer가 추가되어 있어서 주소 매핑을 위한 섹터 번호 제대로 저장이 되었는지 정보가 추가된다.
- ECC : like hash함수 나중에 데이터를 꺼내갈 때 header에 ECC랑 내부 ECC 비교해서 불량 검사가능.
- ECC의 정도에 따라서 에러를 수정할수도 아니면 확인만 할 수도 있음.

> 그리고 나서 섹터 영역들을 묶어서 독립적인 디스크로 만들면 파티션이 됩니다. 운영체제는 Logical Disk를 독립적 디스크로 생각하게 됩니다. → Swap Or DIsk
>
> 이런 디스크에 파일 시스템을 설치하는 것이 피지컬 포메팅.
> 
> Booting에 대해서 간결하게 말하면 CPU는 Disk에 접근을 하지는 못함. ROM에 정보가 유지되고 있고 로더가 실행됨.

참고로, 0 번 섹터는 부팅용이다.

<img width="706" height="407" alt="스크린샷 2025-08-24 오전 1 51 05" src="https://github.com/user-attachments/assets/118cfa77-e4df-409d-aaeb-ceed7e5b8da4" />
<img width="801" height="513" alt="스크린샷 2025-08-24 오전 1 51 18" src="https://github.com/user-attachments/assets/6704c710-5625-4b80-b6b0-7e5f5a0b7fdd" />

<img width="709" height="572" alt="스크린샷 2025-08-24 오전 1 51 33" src="https://github.com/user-attachments/assets/6f5d2282-3abf-45c0-be75-e97b4992f2de" />

- 디스크에 먼저 들어온 요청을 먼저 처리하는 방식
- 최악의 경우 디스크의 한쪽 끝과 반대쪽 끝에 번갈아 도착 > 탐색시간 비효율적으로 늘어남
  
<img width="704" height="577" alt="스크린샷 2025-08-24 오전 1 51 53" src="https://github.com/user-attachments/assets/ed82437f-a3f8-4d5a-8d7f-de1f44c3882c" />

- 헤드의 현재 위치로부터 가장 가까운 위치에 있는 요청을 제일 먼저 처리하는 알고리즘
- 효율성은 증가시키지만 > 기아 현상(starvation) 발생 가능
- 또한 그리디 느낌이라 이동거리 측면에서 가장 우수한 알고리즘은 아님

<img width="702" height="523" alt="스크린샷 2025-08-24 오전 1 52 16" src="https://github.com/user-attachments/assets/04c484db-86fd-4085-a1ff-8215304d2e7e" />

- 헤드가 디스크 원판의 안쪽 끝과 바깥쪽 끝을 오가며, 그 경로에 존재하는 모든 요청을 처리(EX, 실린더 위치 1>200>1>200)
- 엘리베이터와 비슷하쥬? 그래서 엘리베이터 스케줄링 알고리즘(elevator scheduling algorithm)이라고도 부름
- FCFS 처럼 불필요한 헤드의 이동 발생 X > 효율성
- SSTF 처럼 기아 현상 발생 X > 형평성
- 하지만, 실린더 위치에 따른 편차가 존재함. 실린더 중앙 쪽이 안, 가장자리 쪽보다 탐색시간이 절반임

<img width="690" height="504" alt="스크린샷 2025-08-24 오전 1 52 41" src="https://github.com/user-attachments/assets/d6851a25-3161-4bf2-8dd9-0069a852ccbf" />

- 헤드가 디스크 원판의 안쪽 끝과 바깥쪽 끝을 오가며, 그 경로에 존재하는 모든 요청을 처리(EX, 실린더 위치 1>200>1>200)
- 엘리베이터와 비슷하다. 그래서 엘리베이터 스케줄링 알고리즘(elevator scheduling algorithm)이라고도 부름
- FCFS 처럼 불필요한 헤드의 이동 발생 X > 효율성
- SSTF 처럼 기아 현상 발생 X > 형평성
- 하지만, 실린더 위치에 따른 편차가 존재함. 실린더 중앙 쪽이 안, 가장자리 쪽보다 탐색시간이 절반임

<img width="697" height="461" alt="스크린샷 2025-08-24 오전 1 53 33" src="https://github.com/user-attachments/assets/a2be6b4d-49b9-460b-b134-408eeb3b1b5e" />

- N-Scan : arm이 이동하면서 큐에 들어와 있는 것은 처리하고 도중에 들어오는 것들은 나중에 처리. ( 한 방향 이동하는 와중에)
- Look : Scan하고 Scan 의 비효율 약간 개선 
→ 요청이 있던 없던 계속 움직이는데 요청이 없으면 끝까지 안가도 되니까 . 가던 방향에서 요청이 없다면 방향을 바꾸는 방법.
- C-Look : C-Scan의 Look 버전.

<img width="635" height="351" alt="스크린샷 2025-08-24 오전 1 54 07" src="https://github.com/user-attachments/assets/c552a1e0-6f94-481a-9e4e-dcc2788e8b7e" />

<img width="789" height="614" alt="스크린샷 2025-08-24 오전 1 54 27" src="https://github.com/user-attachments/assets/cfe564fc-642c-40b9-9a8e-8e1fd42c90c5" />

<img width="790" height="428" alt="스크린샷 2025-08-24 오전 1 56 17" src="https://github.com/user-attachments/assets/4623bf90-8971-45f9-89d7-f9d4269bef8c" />

<img width="803" height="480" alt="스크린샷 2025-08-24 오전 1 56 26" src="https://github.com/user-attachments/assets/88d86537-d85e-4664-9b4d-da331f249a77" />

<img width="794" height="650" alt="스크린샷 2025-08-24 오전 1 56 34" src="https://github.com/user-attachments/assets/a907b97b-81a2-43ff-a66c-172a264d8adf" />

<img width="839" height="519" alt="스크린샷 2025-08-24 오전 1 56 44" src="https://github.com/user-attachments/assets/04a3bf4c-535c-49f0-b032-5cd29fd6021d" />

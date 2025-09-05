이론을 알아보았으면 이제는 실습을 진행하면서 직접 Kubernetes를 구축해보자.
NCP(Naver Cloud Platform)의 NKS(Naver Kubernetes Service)를 사용해서 실습을 진행하겠다.

# kubectl 이란?
우선 쿠버네티스를 구축하려면 가장 기본이 되는 먼저 CLI도구인 kubectl를 설치해야한다.
> kubectl = Kubernetes + Control
kubectl은 쿠버네티스 API에 명령을 보내 쿠버네티스 클러스터의 컨트롤 플레인과 통신하기 위한 커맨드라인 툴이다.

내 전 포스터에서 클러스터는 kube-api-server를 통해 통신하며 파드를 만들거나 조회하거나 이런 작업을 하고싶으면 kube-api-server에 명령어를 보내서 Control Plane이랑 상호작용 한다고 말했었다.
이를 가능하게 하는것이 kubectl이다. 물론 이것도 근본적으로 REST API 호출을 사용하는 것이어서 직접 호출하는 것도 가능은 하다. 그러나 복잡성 증가, 인증/보안 관리 부담, API 버전 호환성 문제, 에러 처리의 복잡함은 결국 개발 생산성이 저하되고... kubectl을 사용하는게 권장된다.


우리가 원격저장소를 컨트롤할때 git CLI를 사용하는 거랑 같은 원리이다.

## kubectl 설치 방법
> 쿠버네티스 공식 가이드: https://kubernetes.io/docs/tasks/tools/

공식 가이드를 보면 운영체제에 맞게 Linux, MacOS, Windows 각각 설치 방법이 나와있다.
나는 윈도우기 때문에 윈도우 방법으로 진행한다.

한가지 주의해야하는 점은 '클러스터와 마이너 버전 차이가 하나 이내인 kubectl 버전을 사용해야 합니다. 예를 들어, v1.34 클라이언트는 v1.33, v1.34, v1.35 제어 평면과 통신할 수 있습니다.' 라고 가이드에 나와있으며, 이는 내 클러스터 생성 버전이랑 하나 차이 이내 나는 버전을 설치해야 호환문제가 발생하지 않는다.

```
# 1. kubectl v1.31.0 다운로드
curl.exe -LO "https://dl.k8s.io/release/v1.31.0/bin/windows/amd64/kubectl.exe"

# 2. D 드라이브에 설치 디렉토리 생성
mkdir D:\kubectl -Force

# 3. 파일 이동
move kubectl.exe D:\kubectl\

#### 4. PATH에 D:\kubectl 추가 (현재 세션용)
$env:PATH += ";D:\kubectl"

# 5. 영구적으로 PATH 추가 (사용자 환경변수)
[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";D:\kubectl", [EnvironmentVariableTarget]::User)

# 6. 설치 확인
kubectl version --client
```

1. 다운로드하고 kubectl.exe가 드라이브에 설치될 것이다.
2. 특정 저장하고 싶은 위치가 있으면 폴더를 만들고
3. kubectl.exe 를 옮겨준다.
4. 환경 변수를 설정해줘서 kubectl 명령어를 작성했을때 해당 루트를 찾아서 인식하게 설정한다. 4번은 현재 powershell에서 즉시 환경변수 설정해서 재부팅 안해도되고,
5. 5번은 영구적으로 다음에 설정을 안해도 로컬 환경변수에 저장해버린다.
6. 설치가 잘 됐는지 확인해보면 아래와 같이 버전이 잘 나올 것이다.
  
![](https://velog.velcdn.com/images/krewooo/post/c2ae6c16-7f62-4a57-bb00-54334171f15e/image.png)




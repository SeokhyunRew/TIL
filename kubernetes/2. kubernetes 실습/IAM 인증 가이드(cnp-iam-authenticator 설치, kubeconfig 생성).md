이전 포스터에서 kubectl을 설치했다. 해당 실습은 NCP(Naver Cloud Platform)을 사용하여 진행한다고 했는데, 이때 NKS(Naver Kubernetes Service)의 클러스터에 kubectl을 사용해서 접근하려면 권한이 필요하게 된다. 이때 IAM 인증을 사용한다.
> IAM 인증이란? 
IAM(Identity and Access Management) 인증이란 IT 시스템에서 사용자의 신원을 확인(인증)하고, 해당 신원에 부여된 권한에 따라 특정 리소스(데이터, 애플리케이션 등)에 대한 액세스를 허용하거나 거부하는 과정

NKS는 IAM 클러스터 인증을 위해서 ncp-iam-authenticator를 사용합니다. ncp-iam-authenticator를 통해 IAM 인증이 적용된 kubeconfig를 가져와 기존 파일에 업데이트하거나 새 파일로 생성할 수 있습니다.
그래서 이번 실습단계에서는 ncp-iam-authenticator설치와 이를 활용하여 kubeconfig파일을 생성하겠다. 

*참고로 메인 계정은 IAM 인증을 지원하지 않는다. 메인 계정은 모든 리소스와 서비스에 대한 최고 권한을 가지므로 실수로 전체 인프라를 삭제하는 등 치명적인 제어가 가능하다.
그래서 메인 계정은 API 접근 권한을 제어하고 서브계정을 사용하여 kubectl을 통한 API 명령어를 이용 가능하다.
내가 받은 계정도 K-PaaS 클라우드 플랫폼에서 제공받은 서브계정이므로 IAM 인증을 이용한다.*

# ncp-iam-authenticator 설치
> ncp-iam-authenticator 설치 가이드: https://guide.ncloud-docs.com/docs/k8s-iam-auth-ncp-iam-authenticator

해당 가이드에서 windows 설치 가이드 명령어를 활용해서 설치했다.
```
# ncp-iam-authenticator 다운로드
curl.exe -o ncp-iam-authenticator.exe -L https://github.com/NaverCloudPlatform/ncp-iam-authenticator/releases/latest/download/ncp-iam-authenticator_windows_amd64.exe

# 설치 확인
ncp-iam-authenticator help
```
![](https://velog.velcdn.com/images/krewooo/post/dac17c81-403e-478a-a122-7f0efc1e0e41/image.png)

잘 설치된 모습이다.

# API 인증키 생성
우선 해당 클러스터의 kubeconfig를 생성하려면 API 인증키가 필요하다. 왜냐하면 kubeconfig파일을 생성하는 명령어도 API 이기 때문이다.
> NCP 콘솔 > 계정 관리 > 인증키 관리 > 신규 API 인증키 생성 
![](https://velog.velcdn.com/images/krewooo/post/ab279f1b-f24e-4f64-8c6f-4ea49f8fc8f7/image.png)

해당 그림처럼 Access Key Id & Secret Key를 생성해주면 된다.

# ncp-iam-authenticator API 인증키값 설정
생성한 Access Key Id & Secret Key를 OS 환경 변수 Or 사용자 환경 홈 디렉터리의 .ncloud 폴더에 configure 파일 생성 중 하나 하면된다.
둘다 안해도 API 키 값을 직접 입력하면 되나, 나는 OS 환경변수 설정 방법을 택했다.

![](https://velog.velcdn.com/images/krewooo/post/ac4c6716-29cf-4f3a-ac75-bea3daaa61dc/image.png)

```
# 공식 가이드에 따른 환경변수 설정
$env:NCLOUD_ACCESS_KEY = "ACCESS_KEY"
$env:NCLOUD_SECRET_KEY = "SECRET_KEY"
$env:NCLOUD_API_GW = "https://ncloud.apigw.ntruss.com"
```

# kubeconfig 파일 생성
이제 전에 만들었던 ncp-iam-authenticator.exe를 사용해서 kubeconfig 파일을 생성해야 한다. 아래 명령어에 사용될 옵션을 참고해서 생성하면 된다.
![](https://velog.velcdn.com/images/krewooo/post/154ea4e3-486c-45c9-814c-f940b719cd0c/image.png)
```
# kubeconfig를 D:\kubectl에 생성(다 모아둘려고)
D:\kubectl\ncp-iam-authenticator.exe create-kubeconfig --region KR --clusterUuid <클러스터 uuid 입력> --output 저장할 파일 경로 입력

# kubectl이 D:\kubectl\config를 사용하도록 설정
# powershell# KUBECONFIG 환경변수 설정
$env:KUBECONFIG = "아까 입력한 output 파일 경로"

# 영구적으로 설정
[Environment]::SetEnvironmentVariable("KUBECONFIG", "아까 입력한 output 파일 경로", [EnvironmentVariableTarget]::User)
```
이런식으로 설정했다.

# 명령어 TEST
kubeconfig파일로 클러스터의 namespace를 가져오는 kuectl 명령 테스트를 해보자.
![](https://velog.velcdn.com/images/krewooo/post/e7a386c0-8f45-42cd-9a90-0d1f672c2c0e/image.png)

원래 이런식으로 --kubeconfig 의 파일루트를 지정해서 실행해줘야 하는데, 아까 나는 환경변수로 KUBECONFIG에 입력해주어거 kubectl이 알아서 config 파일을 찾는다. 그래서 아래 명령어 처럼 간단히 된다.

![](https://velog.velcdn.com/images/krewooo/post/64188424-2ee5-4b27-8ded-76582d937211/image.png)

namespace를 살펴보자. 따로 생성을 안했기에 default 네임스페이스가 존재하며 kube-node-lease, kube-public, kube-system은 자동으로 생기는 네임스페이스다.
- default: 
  - 일반 사용자 작업 공간
  - Namespace를 지정하지 않은 경우에 기본적으로 객체 생성시 할당되는 Namespace이다.(예: kubectl create deployment my-app → default 네임스페이스에 생성됨)
- kube-node-lease:
  - 노드 상태 관리 전용 공간
  - 각 노드가 "나 살아있어요!"라고 신호를 보내는 곳
  - 노드 하트비트(heartbeat) 정보 저장
- kube-public:
  - 모든 사용자가 읽을 수 있는 공개 공간
  - 클러스터 정보, 인증서 등 공개 정보 저장
  - 인증받지 않은 사용자도 읽기 가능
- kube-system:
  - 쿠버네티스 핵심 시스템 전용 공간
  - DNS, 네트워킹, 모니터링 등 클러스터 운영에 필수적인 구성요소들
  - 사용자가 함부로 건드리면 안 되는 시스템 파드들

> namespace란?
쿠버네티스 클러스터 내의 논리적인 구분 단위이다. 논리적인 구분 단위는 뭘까? 예를들어 워커 노드를 하나 생성했다. 그럼 2개의 워커노드는 물리적인 서버로 나뉘어 물리적 구분 단위이다. 하지만 네임스페이스를 생성하면 실제 물리적으로 생성되는 공간은 아니지만 각각의 오브젝트에 해당 네임스페이스 태그를 달아서 관리가 가능하다.
그래서 한 네임스페이스를 삭제한다면 그 네임스페이스 태그를 단 객체들이 모두 삭제가 된다. 네이스페이스는 주로 효과적으로 자원을 관리할때 사용된다. 예를들어 한 노드 안에 test랑 real이 다있다면 test 네임스페이스랑 real 네임스페이스를 따로두어 test 네임서버에서 실행되 객체들이 가용할 수 있는 CPU, Memory 양을 30%만 설정하고, 실제 운영하는 real 환경이 나머지 자원을 70%를 할당하며 효과적으로 클러스터를 관리할 수 있도록 도와준다.   

- 클러스터의 노드와 정보도 확인해보자
  - 워커 노드2개가 활성화(Ready) 돼있고 만들어 진지(AGE) 각각 13분, 28일 됐다.
  - 클러스터의 정보에는 클러스터 엔드포인트가 나와있다.(저 마스킹 된 부분이 클러스터 uuid이다). 이 uuid를 포함한 클러스터 엔드포인트로 kubectl이 명령을 보내면 클러스터 컨트롤 플레인의 kube-api-server와 상호작용 한다!

> UUID란?
UUID는 'Universally Unique Identifier'의 약자로 128-bit의 고유 식별자에요. 다른 고유 ID 생성 방법과 다르게 UUID는 중앙 시스템에 등록하고 발급하는 과정이 없어서 상대적으로 더 빠르고 간단하게 만들 수 있다는 장점이 있어요.
 
![](https://velog.velcdn.com/images/krewooo/post/704c80b1-474a-46b5-a853-8390168575b5/image.png)

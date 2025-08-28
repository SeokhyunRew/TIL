Kubernetes의 구조를 살펴보고 구성요소에 대해 알아보자.

# Kubernetes Cluster 아키텍쳐

## Cluster란?
일단 쿠버네티스 생태계를 구축하기 위해서 클러스터를 구축해야한다.

쿠버네티스 아키텍처에서 클러스터(Cluster)란 컨테이너 형태의 애플리케이션을 호스팅하는 물리/가상 환경의 노드들로 이루어진 집합이다.

그래서 쉽게 이해하면 클러스터 하나마다 쿠버네티스 생태계가 구축된 집합이라고 볼 수 있다. 아래 그림은 쿠버네티스 클러스터 1개의 아키텍쳐이다. 클러스터 여러개를 둘 수도 있다.

## Cluster Architecture
일단 클러스터 아키텍쳐의 큰 구조, 데이터를 주고 받는 방법에 대해서 설명하겠다.

![](https://velog.velcdn.com/images/krewooo/post/d2810255-9e88-4fc9-9253-56a9950b8894/image.png)

### Control Plane & Node
그림을 보면 기본적으로 Cluster 안에는 Control Plane 과 여러개의 Node들로 구성돼 있다.
Control Plane은 다른 노드들을 관리하고 제어하는 역할을 하는 집합으로 논리적인 개념이다.
일반적으로 Control Plane을 실행하는 서버/노드를 Master Node라고 하며, 이전에 설명한 다른 노드 들을 Worker Node 라고 부른다.

그래서 그림의 Control Plane 섹션은 Master Node에서, 다른 Node1, 2 는 각각의 Worker Node에서 실행된다.

### Cloud-Controller-Manager(CCM)
방금 Control Plane에서 다른 노드들을 관리한다고 했는데, 이때 감시자(통역사)는 cloud-controller-manager이다.
ccm은 LoadBalancer Service, PersistentVolume 등이 생성되거나 변경되면 이를 감지하고 클라우드 리소스 요청으로 번역해서 Cloud Provider API에게 전달한다.  

### Cloud Provider API
그럼 Cloud Provider API는 각 플랫폼 별 정의돼 있는 실제 명령을 수행할 API를 실행한다.

### kube-api-server
또한, 클러스터의 노드 간에는 api로 통신한다.
클러스터의 모든 통신은 kube-api-server를 중심으로 이루어진다. Master Node의 모든 컴포넌트들(etcd, Controller Manager, Scheduler, CCM) 간의 통신뿐만 아니라, Worker Node의 kubelet과의 통신도 모두 kube-api-server를 통해 API로 이루어진다.


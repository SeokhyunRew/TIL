- 앞서 전 포스터에서 클러스터 구조에 대해서 설명했었다. 하나의 클러스터에는 여러개의 노드(각 노드는 하나의 물리적 서버)가 있고 각 노드 에는 여러개의 파드가 존재한다. 이 파드 하나동 보통 하나의 컨테이너 이미지를 올리는 것이다(하나의 파드 안에 여러개의 컨테이너를 만들때도 있다.).
결론적으로 우리가 애플리케이션이나 이런 도커로 만든 이미지를 실행하고 싶으면 Pod안의 컨테이너에 이미지를 실행한다.
- 해당 계정은 이미 Cluster와 Master Node(자동으로 만들어지면 NCP가 알아서 관리)와 Woker Node를 제공 받았다. 그래서 해당 워커노드에 파드를 구축하고 해당 파드의 컨테이너에 이미지를 실행하면 된다.
- 그럼 이미지는 어디서 가져오냐? 해당 컨테이너에 직접 이미지를 넣을 수 없다. 컨테이너는 이미 실행 중인 상태(쿠버네티스의 선언적 특성)이고, 쿠버네티스가 파드를 생성할 때 미리 정의된 이미지를 기반으로 컨테이너를 실행하기 때문에 Container Registry에 이미지를 저장해두고 이 이미지 저장소에서 필요한 이미지를 pull 해서 사용한다. 그러나 Container Registry는 논리적 저장소이다. Container Registry안에는 이미지 이름을 저장하고 해당 물리적 데이터는 Bucket에 저장하는데 NCP에서는 Object Storage라고 부른다.
- 정리하자면, 현재 Cluster와 WokerNode(서버)가 만들어져 있고 내가 원하는 애플리케이션, MySQL, Redis를 실행시키려면 각각의 파드에 Docker 이미지로 실행해야하며, 이때 Container Registry를 이용해서 이미지를 받아와야 하며, Container Registry를 이용하기 위해서는 물리적 저장소 Object Storage(Bucket)를 만들어야 한다.
> Object Storage의 사용 용도는 다양하다.
> - Docker Hub와 같은 Public 저장소 대신 Private 저장소를 사용하고 싶을때
> - 수집한 로그를 저장
> - DB 백업(Pod가 죽으면 데이터가 다 날라가기 때문에 Persistance Volumn 설정하면 여기에 저장된다)
> - 정적 파일 서빙
> - Config 파일 관리

# Object Storage(Bucket) 생성
1. NCP 콘솔의 VPC/Object Storage/Bucket Management에 접속하여 버킷 생성 버튼을 누른다.
![](https://velog.velcdn.com/images/krewooo/post/6f654ac4-6ae1-4e92-b0d1-a73c6a93700b/image.png)

2. Bucket 이름을 입력
![](https://velog.velcdn.com/images/krewooo/post/55436cce-1250-4ade-9e48-3418306d7891/image.png)

3. 설정관리, 일단 둘다 당장 필요하지 않기에 설정 안하고 넘어가겠다.
- 잠금 설정: 설정한 기간동안 버킷이 삭제되지않고 계약 해지가 불가능 하다
- 암호화 관리: 한번 설정하면 해제할 수 없고 버킷에 저장되는 모든 파일이 자동으로 암호화됨. 권한이 없으면 파일 내용을 볼 수 없음
![](https://velog.velcdn.com/images/krewooo/post/08d40001-5d0b-4e81-8282-73f8b62e6ab2/image.png)

4. 권한 관리
- 공개 선택 시 버킷에 저장된 폴더/파일 목록이 공개된다. 각 파일별로 공개 여부가 설정 가능하기 때문에 db와 같은 정보도 버킷에 있다면 공개 안함으로 해야함.
![](https://velog.velcdn.com/images/krewooo/post/ca1ce2cd-eae5-4346-9041-a11dab6b1f57/image.png)

5. 버킷 생성
![](https://velog.velcdn.com/images/krewooo/post/a2374071-cfa8-4281-a7e5-194c1914bfc9/image.png)


# Container Registry 생성
다음 Container Registry를 생성해보겠다.

1. NCP 콘솔의 VPC/Container Registry에서 레지스트리 생성 버튼을 누른다.
![](https://velog.velcdn.com/images/krewooo/post/dc53cbec-2486-447a-8d04-dd6f1c56c370/image.png)

2. 새로운 레지스트리 추가
- 레지스트리 이름을 입력
- 아까 생성한 버킷 할당(이미 할당된 버킷은 할당할 수 없다.)
![](https://velog.velcdn.com/images/krewooo/post/42fb523a-1fe2-4c84-aad0-276d80c87e00/image.png)

# Container Registry 접근
- Container Registry 정보 확인 및 관리 가이드: https://guide.ncloud-docs.com/docs/container-ncr-1-3
다음은 생성한 Container Registry의 정보 확인 및 관리 방법

1. Docker CLI에 로그인
이전 포스터에서 계정 관리에서 생성한 API Access Key와 API Secret Key를 사용하여 Container Registry에 로그인 해야한다. 로그인 해야지 시크릿 값을 생성하거나 해당 Container Registry에 push 할 수 있다.
```
$ docker login -u <access-key-id> <registry-name>.<region-code>.ncr.ntruss.com
Password: <secret-key>
Login Succeeded

```
![](https://velog.velcdn.com/images/krewooo/post/f6bc11f9-2fdc-4af8-99e3-e984c7282f41/image.png)


2. Container Registry에 접속하기 위한 Secret 값 생성
쿠버네티스에서 Private Container Registry의 이미지를 pull하기 위해서 Container Registry의 Secret 값을 생성해준다. 생성된 Secret 값은 쿠버네티스 클러스터의 etcd(Secret 오브젝트)에 저장된다.
```
$ kubectl create secret docker-registry regcred --docker-server=<registry-end-point> --docker-username=<access-key-id> --docker-password=<secret-key> --docker-email=<your-email>
```
![](https://velog.velcdn.com/images/krewooo/post/dddd10df-ee7f-4d1d-b4ce-b16df91e82a4/image.png)

3. Pod를 배포할 YAML 파일을 작성할때 template의 spec 영역의 imagePullSecrets에 앞서 생성한 secret 이름을 입력해 저장해줘야 한다. 아래는 예시 코드이다.
보면 spec.template.imagePullSecrets.name에 아까 생성한 Secrets의 regcred를 입력해주었고, 이렇게 입력하면 파드 배포시 etcd의 Secrets를 활용하여 Registry를 인증한다.
```
 apiVersion: apps/v1
 kind: Deployment
 metadata:
   name: test-mysql
   labels:
     name: test-mysql
 spec:
   replicas: 1
   selector:
     matchLabels:
       name: test-mysql
   template:
     metadata:
       labels:
         name: test-mysql
     spec:
       imagePullSecrets:
       - name: regcred
       containers:
       - name: test-mysql
         image: <registry-name>.ncr.ntruss.com/mysql:5.7.21
         env:
         - name: MYSQL_ROOT_PASSWORD
           value: "1234"

```

> 다음 포스터에서는 이를 활용해서 Pod에 Container를 생성하여 Docker 이미지를 실행 해보겠다.

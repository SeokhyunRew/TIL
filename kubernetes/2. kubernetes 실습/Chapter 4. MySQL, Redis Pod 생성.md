애플리케이션 Pod를 배포하기 전에 해당 애플리케이션에서 사용하고 있는 DB와 Cache DB인 MySQL과 Redis Pod를 각각 먼저 띄어 보겠다.
앞서 Kubernets YAML 파일을 작성하는 방법에 대해서 알아보고 가자.
# Kubernets YAML 파일 작성 법
Kubernetes YAML 파일의 4가지 핵심 필드인 apiVersion, kind, metadata, spec이 있다. 각가의 필드를 살펴보자.

>1. apiVersion - API 버전 지정
```
apiVersion: v1           # 코어 API 그룹 (Pod, Service, ConfigMap 등)
apiVersion: apps/v1      # apps 그룹 (Deployment, ReplicaSet 등)
apiVersion: storage.k8s.io/v1  # 스토리지 관련
```
쿠버네티스 API는 계속 발전하므로 어떤 버전의 API를 사용할지 명시해야 합니다.

>2. kind - 리소스 타입 지정
```
kind: Pod                # 파드 생성
kind: Service            # 서비스 생성
kind: Deployment         # 디플로이먼트 생성
kind: ConfigMap          # 설정맵 생성
kind: Secret             # 시크릿 생성
```
어떤 종류의 쿠버네티스 객체를 만들 것인지 정의합니다.

> 3. metadata - 객체의 메타데이터
```
metadata:
  name: mysql-deployment     # 객체 이름 (필수)
  namespace: default         # 네임스페이스 (생략 시 default)
  labels:                    # 라벨 (선택사항)
    app: mysql
    tier: database
  annotations:               # 어노테이션 (선택사항)
    description: "MySQL database"
```
객체를 식별하고 분류하는 정보들입니다.

>4. spec - 객체의 원하는 상태 정의
```
# Pod의 spec
spec:
  containers:
  - name: mysql
    image: mysql:8.0
# Service의 spec  
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
# Deployment의 spec
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mysql
```
실제로 어떻게 동작해야 하는지에 대한 상세 설정입니다.

# etcd에 Secret 정보 저장
> 초반 쿠버네티스 컴포넌트 포스터에서 etcd에 대해서 알아봤지만, 한번 더 짚고 가겠다.
etcd란? 
- 분산 키-값 저장소로 모든 클러스터의 모든 상태 정보를 저장하는 쿠버네티스 뒷단의 저장소로 사용되는 일관성·고가용성 키-값 저장소.
- 그렇다면 전에 만들었던 Object Storage랑 뭐가 다른가? 혹시 모르니 이도 비교하고 넘어간다.
  - 가장 큰 차이점은 etcd는 클러스터안에 마스터노드에 존재한다. 그리고 Object Storage는 쿠버네티스가 아닌 별도의 NCP의 데이터를 저장하는 서비스이다.
  - 주로 etcd에서는 해당 클러스터에서 필요한 핵심 정인 Secrete 값들을 빠르게 찾기위해 저장하며, Bucket는 이미지 같이 대용량 파일이나 데이터를 저장한다.
  ![](https://velog.velcdn.com/images/krewooo/post/92e327b8-6132-4ed4-a477-4b001e359792/image.png)

그래서 이런 etcd에 MySQL과 Redis에 필요한 password 같은 값들을 저장한다. Public Git Repository나 코드 저장소에서 mysql.yaml 파일에 접근하더라도 Secret값은 etcd에 있어서 실제 패스워드를 볼 수 없다.  
아래는 예시 Secret.yaml 파일이다. 설정파일은 Config로 따로 빼고 Secret 정보만 담자.
```
# secrets.yaml - 모든 민감한 정보를 Secret으로 관리
apiVersion: v1
kind: Secret
metadata:
  name: database-secrets
  labels:
    app: database
type: Opaque
stringData:  # stringData 사용하면 자동으로 base64 인코딩됨
  # MySQL 인증 정보
  mysql-root-password: "RootPassword"
  mysql-user: "User"
  mysql-password: "UserPassword"
  mysql-database: "DatabaseName"
  
  # Redis 인증 정보  
  redis-password: "RedisPassword"
```

생성 코드
```
# 1. Secret 먼저 생성 (MySQL, Redis가 의존)
kubectl apply -f "secrets.yaml"
```
# MySQL Pod 생성
> PVC: 스토리지 설정 및 접근 모드 설명
ConfigMap: MySQL 성능 최적화 설정들의 각 항목별 설명
Deployment: 환경변수, 리소스 제한, Health Check 등 상세 설명
Service: 네트워크 접근 방식 설명

MySQL 예시 YAML

```
# PersistentVolumeClaim - MySQL 데이터 영구 저장용 스토리지
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  labels:
    app: mysql
spec:
  accessModes:
    - ReadWriteOnce  # 하나의 노드에서만 읽기/쓰기 가능
  resources:
    requests:
      storage: 20Gi  # 20GB 스토리지 요청
  storageClassName: storageClassName  # 사용할 스토리지

---
# ConfigMap - MySQL 설정 파일 저장
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
  labels:
    app: mysql
data:
  my.cnf: |
    [mysqld]
    # 기본 엔진 및 SQL 모드 설정
    default-storage-engine=InnoDB
    sql_mode=NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
    
    # 문자셋 설정 (한글, 이모지 지원)
    character-set-server=utf8mb4
    collation-server=utf8mb4_unicode_ci
    
    # InnoDB 성능 최적화
    innodb_buffer_pool_size=512M     # 메모리 버퍼 크기
    innodb_log_file_size=128M        # 로그 파일 크기
    innodb_flush_log_at_trx_commit=2 # 트랜잭션 커밋 시 로그 플러시 방식
    innodb_file_per_table=1          # 테이블별 개별 파일 사용
    
    # 연결 및 성능 설정
    max_connections=200              # 최대 동시 연결 수
    max_connect_errors=100000        # 최대 연결 오류 허용 수
    
    # 로깅 설정
    slow_query_log=1                 # 느린 쿼리 로그 활성화
    slow_query_log_file=/var/log/mysql/slow.log
    long_query_time=2                # 2초 이상 쿼리를 느린 쿼리로 기록
    
    # 복제 설정
    log-bin=mysql-bin                # 바이너리 로그 활성화
    binlog_format=ROW                # 바이너리 로그 형식
    binlog_expire_logs_seconds=604800 # 바이너리 로그 보관 기간 (7일)
    
    # 시간대 및 기타 설정
    default-time-zone='+09:00'       # 한국 시간대
    tmp_table_size=64M               # 임시 테이블 크기
    max_heap_table_size=64M          # 메모리 테이블 최대 크기
    default_authentication_plugin=mysql_native_password  # 기본 인증 방식
    general_log=0                    # 일반 로그 비활성화
    log_error_verbosity=2            # 에러 로그 상세 수준
    
    [mysql]
    default-character-set=utf8mb4    # 클라이언트 기본 문자셋
    
    [client]
    default-character-set=utf8mb4    # 클라이언트 기본 문자셋

---
# Deployment - MySQL 컨테이너 배포 설정
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: mysql
    tier: database
spec:
  replicas: 1  # MySQL은 보통 단일 인스턴스로 실행
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
        tier: database
    spec:
      containers:
      - name: mysql
        image: mysql:8.0  # MySQL 8.0 이미지 사용
        ports:
        - containerPort: 3306  # MySQL 기본 포트
        
        # 환경변수 - Secret에서 민감한 정보 가져오기
        env:
        - name: MYSQL_ROOT_PASSWORD  # root 계정 패스워드
          valueFrom:
            secretKeyRef:
              name: database-secrets
              key: mysql-root-password
        - name: MYSQL_DATABASE      # 기본 생성할 데이터베이스명
          valueFrom:
            secretKeyRef:
              name: database-secrets
              key: mysql-database
        - name: MYSQL_USER         # 일반 사용자 계정
          valueFrom:
            secretKeyRef:
              name: database-secrets
              key: mysql-user
        - name: MYSQL_PASSWORD     # 일반 사용자 패스워드
          valueFrom:
            secretKeyRef:
              name: database-secrets
              key: mysql-password
        
        # 리소스 제한 설정
        resources:
          requests:
            memory: "800Mi"  # 최소 메모리 요구사항
            cpu: "400m"      # 최소 CPU (0.4 코어)
          limits:
            memory: "1Gi"    # 최대 메모리 제한
            cpu: "600m"      # 최대 CPU (0.6 코어)
        
        # 볼륨 마운트 설정
        volumeMounts:
        - name: mysql-storage  # 데이터 영구 저장용
          mountPath: /var/lib/mysql
        - name: mysql-config   # 설정 파일 마운트
          mountPath: /etc/mysql/conf.d
        
        # 생존 상태 확인 (컨테이너가 정상 작동하는지 체크)
        livenessProbe:
          exec:
            command:
            - /bin/bash
            - -c
            - mysqladmin ping -h localhost -u root -p$MYSQL_ROOT_PASSWORD
          initialDelaySeconds: 60  # 시작 후 60초 대기
          periodSeconds: 30        # 30초마다 확인
          timeoutSeconds: 10       # 10초 타임아웃
          failureThreshold: 3      # 3번 실패 시 컨테이너 재시작
        
        # 준비 상태 확인 (트래픽을 받을 준비가 되었는지 체크)
        readinessProbe:
          exec:
            command:
            - /bin/bash  
            - -c
            - mysqladmin ping -h localhost -u root -p$MYSQL_ROOT_PASSWORD
          initialDelaySeconds: 30  # 시작 후 30초 대기
          periodSeconds: 10        # 10초마다 확인
          timeoutSeconds: 5        # 5초 타임아웃
          failureThreshold: 3      # 3번 실패 시 트래픽 차단
      
      # 볼륨 정의
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc  # 위에서 정의한 PVC 사용
      - name: mysql-config
        configMap:
          name: mysql-config    # 위에서 정의한 ConfigMap 사용

---
# Service - 네트워크 접근 설정
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  labels:
    app: mysql
spec:
  selector:
    app: mysql  # app=mysql 라벨을 가진 Pod들에게 트래픽 전달
  ports:
  - name: mysql
    port: 3306        # 서비스 포트 (다른 Pod에서 접근할 때 사용)
    targetPort: 3306  # 컨테이너 포트 (실제 MySQL이 리스닝하는 포트)
  type: ClusterIP     # 클러스터 내부에서만 접근 가능
```
그리고 만든 mysql.yaml을 배포하자.
```
# 2. MySQL 배포  
kubectl apply -f "mysql.yaml"
```

# Redis Pod 생성
redis.yaml 파일 생성 예시
```
# ConfigMap - Redis 설정 파일 저장
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-config
  labels:
    app: redis
data:
  redis.conf: |
    # 네트워크 설정
    bind 0.0.0.0              # 모든 네트워크 인터페이스에서 연결 허용
    port 6379                 # Redis 기본 포트
    timeout 0                 # 클라이언트 연결 타임아웃 (0=무제한)
    tcp-keepalive 300         # TCP keepalive 설정 (초)
    
    # 프로세스 설정
    daemonize no              # 백그라운드 실행 안함 (컨테이너에서는 no)
    supervised no             # 시스템 관리자 없음
    pidfile /var/run/redis_6379.pid  # PID 파일 위치
    
    # 로깅 설정
    loglevel notice           # 로그 레벨 (debug/verbose/notice/warning)
    logfile ""                # 로그 파일 (빈 문자열=stdout)
    databases 16              # 데이터베이스 개수 (0-15번)
    
    # 메모리 관리
    maxmemory 300mb           # 최대 메모리 사용량
    maxmemory-policy allkeys-lru  # 메모리 부족 시 LRU 알고리즘으로 삭제
    maxmemory-samples 5       # LRU 샘플링 개수
    
    # 데이터 영속성 설정 (RDB 스냅샷)
    save 900 1                # 900초 동안 1개 이상 변경 시 저장
    save 300 10               # 300초 동안 10개 이상 변경 시 저장
    save 60 10000             # 60초 동안 10000개 이상 변경 시 저장
    stop-writes-on-bgsave-error yes  # 백그라운드 저장 실패 시 쓰기 중단
    rdbcompression yes        # RDB 파일 압축 사용
    rdbchecksum yes           # RDB 파일 체크섬 사용
    dbfilename dump.rdb       # RDB 파일명
    dir /data                 # 데이터 파일 저장 경로
    
    # 복제 설정 (Master-Slave)
    replica-serve-stale-data yes  # 복제본이 오래된 데이터 제공 허용
    replica-read-only yes     # 복제본을 읽기 전용으로 설정
    
    # 보안: 위험한 명령어 비활성화
    rename-command FLUSHDB ""     # 데이터베이스 전체 삭제 명령 비활성화
    rename-command FLUSHALL ""    # 모든 데이터베이스 삭제 명령 비활성화
    rename-command DEBUG ""       # 디버그 명령 비활성화
    
    # 클라이언트 및 성능 설정
    maxclients 100            # 최대 클라이언트 연결 수
    slowlog-log-slower-than 10000  # 10ms 이상 쿼리를 느린 쿼리로 기록
    slowlog-max-len 128       # 느린 쿼리 로그 최대 개수
    latency-monitor-threshold 100  # 지연 모니터링 임계값 (ms)
    
    # 데이터 구조 최적화
    hash-max-ziplist-entries 512   # Hash 압축 최대 엔트리 수
    hash-max-ziplist-value 64      # Hash 압축 최대 값 크기
    list-max-ziplist-size -2       # List 압축 크기
    set-max-intset-entries 512     # Set 정수 집합 최대 엔트리
    zset-max-ziplist-entries 128   # Sorted Set 압축 최대 엔트리
    
    # 기타 설정
    activerehashing yes       # 백그라운드 해시 테이블 재구성 활성화
    hz 10                     # 백그라운드 작업 빈도
    dynamic-hz yes            # 동적 빈도 조정
    aof-rewrite-incremental-fsync yes  # AOF 재작성 시 점진적 동기화

---
# Deployment - Redis 컨테이너 배포 설정
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  labels:
    app: redis
    tier: cache             # 캐시 계층임을 표시
spec:
  replicas: 1               # Redis는 보통 단일 인스턴스로 실행
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
        tier: cache
    spec:
      containers:
      - name: redis
        image: redis:7-alpine    # Redis 7.x Alpine Linux 경량 이미지
        ports:
        - containerPort: 6379    # Redis 기본 포트
        
        # Redis 서버 시작 명령어 (패스워드 포함)
        command:
        - redis-server
        - --requirepass           # 패스워드 인증 요구
        - $(REDIS_PASSWORD)       # 환경변수에서 패스워드 가져오기
        - --maxmemory
        - "300mb"                # 최대 메모리 300MB로 제한
        - --maxmemory-policy
        - "allkeys-lru"          # 메모리 부족 시 LRU 정책 사용
        
        # 환경변수 - Secret에서 민감한 정보 가져오기
        env:
        - name: REDIS_PASSWORD   # Redis 접속 패스워드
          valueFrom:
            secretKeyRef:
              name: database-secrets
              key: redis-password
        
        # 리소스 제한 설정
        resources:
          requests:
            memory: "200Mi"      # 최소 메모리 요구사항
            cpu: "100m"          # 최소 CPU (0.1 코어)
          limits:
            memory: "400Mi"      # 최대 메모리 제한
            cpu: "300m"          # 최대 CPU (0.3 코어)
        
        # 볼륨 마운트 설정
        volumeMounts:
        - name: redis-data       # 데이터 저장용 볼륨
          mountPath: /data       # Redis 데이터 디렉토리
        
        # 생존 상태 확인 (Redis가 정상 작동하는지 체크)
        livenessProbe:
          exec:
            command:
            - /bin/sh
            - -c
            - redis-cli -a $REDIS_PASSWORD ping  # PING 명령으로 상태 확인
          initialDelaySeconds: 30   # 시작 후 30초 대기
          periodSeconds: 10         # 10초마다 확인
          timeoutSeconds: 5         # 5초 타임아웃
          failureThreshold: 3       # 3번 실패 시 컨테이너 재시작
        
        # 준비 상태 확인 (트래픽을 받을 준비가 되었는지 체크)
        readinessProbe:
          exec:
            command:
            - /bin/sh
            - -c  
            - redis-cli -a $REDIS_PASSWORD ping  # PING 명령으로 준비 상태 확인
          initialDelaySeconds: 5    # 시작 후 5초 대기
          periodSeconds: 5          # 5초마다 확인
          timeoutSeconds: 3         # 3초 타임아웃
          failureThreshold: 3       # 3번 실패 시 트래픽 차단
      
      # 볼륨 정의
      volumes:
      - name: redis-data
        emptyDir: {}             # 임시 디스크 볼륨 (Pod 삭제 시 데이터 사라짐)
                                # 영구 저장이 필요하면 PVC 사용 권장

---
# Service - 네트워크 접근 설정
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  labels:
    app: redis
spec:
  selector:
    app: redis              # app=redis 라벨을 가진 Pod들에게 트래픽 전달
  ports:
  - name: redis
    port: 6379              # 서비스 포트 (다른 Pod에서 접근할 때 사용)
    targetPort: 6379        # 컨테이너 포트 (실제 Redis가 리스닝하는 포트)
  type: ClusterIP           # 클러스터 내부에서만 접근 가능
```
이제 생성한 redis.yaml로 배포해보자
```
# 3. Redis 배포
kubectl apply -f "redis.yaml"
```

# 결과 확인
![](https://velog.velcdn.com/images/krewooo/post/91fd28f5-c8e7-4b3e-ba62-3d1d6911733a/image.png)

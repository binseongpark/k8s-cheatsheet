# k8s-cheatsheet
k8s 커맨드를 기록해두자


# docker

## docker clean up
```
docker rm $(docker ps -aq) -f
docker rmi $(docker images -q) -f
```

# kubernetes

## 컨테이너 실행
```
kubectl run mynginx --image nginx
```

## pod 삭제
```
kubectl delete pod mynginx
```

## 템플릿 파일 생성
```
kubectl run mynginx --image=nginx --dry-run=client -o yaml > mynginx.yaml
```

## pod 의 상태를 확인해야 할때
```
kubectl describe pod mynginx
```

## pod 상세 조회(노드, ip 등)
```
kubectl get pod -o wide
```

## pod 의 이미지 교체하기
```
kubectl edit pod redis
```
이후 vi 편집기에서 `-image` 수정 `:wq` 으로 나오게 되면 자동으로 이미지를 내려받고 업데이트 됨

## replicatset 의 api version
```
apps/v1
```

## 쿠버네티스 모든 리소스 조회
```
kubectl api-resources
```
해당 커맨드로 리소스의 version apiVersion, kind 을 어떤 것을 기입해야 하는지 확인 가능

## yaml 파일 싫행. 선언형 명령 정의서(YAML) 기반의 컨테이너 생성
```
kubectl apply -f myreplicatset.yaml
```

## replicatset 이미지 수정 후 새로운 이미지를 올라오게 하려면
replicatset 을 삭제하거나 모든 pod 를 삭제하면 새로운 이미지의 파드로 올라오게 된다. edit 만 한다고 해서 기존 pod 가 terminate 되지 않음

## replicatset 개수 변경
```
kubectl scale rs --replicas=5 new-replica-set
kubectl edit rs new-replica-set
```
scale 을 이용하던지 edit 을 이용하여 `spec.replicas` 를 수정하면 됨

## yaml 정의서 작성 주의 사항
- kind 대소문자 구문 확실히! deployment x -> Deployment
```
kubectl api-resources
```

## 전체 네임스페이스의 pod 조회
```
kubectl get pod --all-namespaces
```

## 특정 네임스페이스에 pod 생성
```
kubectl run nginx --image=nginx -n namespace
k run redis --image=redis -n finance
```

## service 도메인 주소 법칙
`<서비스 이름>.<네임스페이스>.svc.cluster.local`

## service 항목 정리
port: Service 로 들어오는 포트를 지정
protocol: TCP, UDP, HTTP 등
targetPort: 트래픽을 전달할 컨테이너의 포트를 지정
nodePort: 호스트의 포트 지정

:30080(nodePort) -> :8080(port) -> :80(targetPort)

30080 으로 클러스터로 들어오면 8080 서비스로 가서 80 을 연결해준다

## 라벨 부여하기
```
kubectl label pod redis tier=db
```

## pod 만들면서 service 만들기
```
kubectl run custom-nginx --image=nginx --expose --port 8080 --dry-run=client -o yaml > custom-nginx.yaml
```

## namespace 생성
```
kubectl create namepscae dev-ns
```

## 특정 namespace 에 yaml 실행
```
kubectl apply -f redis-deploy.yaml -n dev-ns
```

## 노드 조회 시 라벨도 표시하기
```
kubectl get node --show-labels
```

## 노드에 라벨 부여하기
```
kubectl label node node01 disktype=ssd
```

## 노드에 taint 제거하기
node-role.kubernetes.io/master 에 담겨있는 taint 값을 없앤다
```
kubectl taint node controlplane node-role.kubernetes.io/master-
```

## 특정 노드에 파드 실행시키기
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: foo-node # 특정 노드에 파드 스케줄
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

## 특정라벨 파드조회
```
kubectl get pod -l bu=finance
```

## 여러개 라벨 파드 조회
```
kubectl get pod -l bu=finance,tier=frontend,env=prod
```

## 전체 리소스에서 특정 라벨 조회
```
kubectl get all --selector env=prod
```

## 전체 리소스에서 여러개 라벨 조회
```
kubectl get all --selector env=prod,bu=finance,tier=frontend
```

## 노드에 taint 적용하기
```
kubectl taint node node01 spray=mortein:NoSchedule
```

## 파드에 toleration 적용하기
```
apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
  - image: nginx
    name: nginx
  tolerations:
  - key: "spray"
    value: "mortein"
    operator: "Equal"
    effect: "NoSchedule"
```

## nodeAffinity key만 있으면 될 때
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      name: red
  template:
    metadata:
      labels:
        name: red
    spec:
      containers:
      - name: nginx
        image: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
```
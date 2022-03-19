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
해당 커맨드로 리소스의 version apiVersion 을 어떤 것을 기입해야 하는지 확인 가능

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
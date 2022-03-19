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
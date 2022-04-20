pod 생성
```
kubectl apply -f pod.yaml
```

pod 목록 확인
```
kubectl get pod
NAME      READY   STATUS    RESTARTS   AGE
new-pod   1/1     Running   0          44h
```

pod 삭제
```
kubectl delete pod po-nginx
pod "po-nginx" deleted
```

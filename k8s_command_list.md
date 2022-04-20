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
deployset 목록
```
kubectl get deployments.apps
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
deploy-nginx   3/3     3            3           22m
```
replicaset 목록
```
kubectl get replicasets.apps
NAME                     DESIRED   CURRENT   READY   AGE
deploy-nginx-9995864d4   3         3         3       21m
rs-nginx                 3         3         3       50s
```
deploy 삭제
```
kubectl delete deploy deploy-nginx
deployment.apps "deploy-nginx" deleted
```
replicaset 삭제
```
kubectl delete replicasets.apps rs-nginx
replicaset.apps "rs-nginx" deleted
```

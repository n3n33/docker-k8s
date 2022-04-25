## 분석 환경 구성도

elasticsearch rolling up
```
kubectl set image deployments/elasticsearch elasticsearch=elastic/elasticsearch:7.17.1
kubectl describe pods
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  84s   default-scheduler  Successfully assigned default/elasticsearch-584b4874bb-zs45d to test2.com
  Normal  Pulling    82s   kubelet            Pulling image "elastic/elasticsearch:7.17.1"
  Normal  Pulled     41s   kubelet            Successfully pulled image "elastic/elasticsearch:7.17.1" in 41.703260989s
  Normal  Created    41s   kubelet            Created container elasticsearch
  Normal  Started    40s   kubelet            Started container elasticsearch
```

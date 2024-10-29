## 查询 ValidatingWebhookConfiguration
```
kubectl get validatingwebhookconfiguration 
```

## 1.禁用 Istio 的 ValidatingWebhookConfiguration
```
kubectl delete validatingwebhookconfiguration istiod-default-validator  
kubectl delete validatingwebhookconfiguration istio-validator-istio-system
```

## filters查询
```
[root@master filter]# kubectl get envoyfilters -n istio-system
NAME         AGE
rsa-filter   11m
```


##  envoyfilter详情
```
kubectl describe envoyfilter rsa-filter -n istio-system
```

## 查看envoy监听哪些filter
```
istioctl proxy-config listeners istio-ingressgateway-<pod-name> -n istio-system
```

## 生效rsafitler
```
kubectl delete envoyfilter rsa-filter -n istio-system
kubectl apply -f rsa.yaml
kubectl delete pod <ingressgateway-pod-name> -n istio-system
```
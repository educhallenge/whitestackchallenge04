# PASO 5 DE CHALLENGE 04 : CREAR HPA USANDO LA MÉTRICA EXTERNA

## VERIFICAR API SERVICE

El API Service v1beta1.custom.metrics.k8s.io es importante porque permitirá que el HPA (lo cual se desplegará en  el paso 5) pue
se comunica correctamente con el servicio prometheus-adapter como se ve a continuación:

```
ubuntu@ubuntu:~$ kubectl get apiservices | grep "prometheus\|NAME"
NAME                                     SERVICE                         AVAILABLE   AGE
v1beta1.custom.metrics.k8s.io            monitoring/prometheus-adapter   True        48m
```
```
ubuntu@ubuntu:~$ kubectl describe apiservices v1beta1.custom.metrics.k8s.io
Name:         v1beta1.custom.metrics.k8s.io
Namespace:
Labels:       app.kubernetes.io/component=metrics
              app.kubernetes.io/instance=prometheus-adapter
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=prometheus-adapter
              app.kubernetes.io/part-of=prometheus-adapter
              app.kubernetes.io/version=v0.11.2
              helm.sh/chart=prometheus-adapter-4.10.0
Annotations:  meta.helm.sh/release-name: prometheus-adapter
              meta.helm.sh/release-namespace: monitoring
API Version:  apiregistration.k8s.io/v1
Kind:         APIService
Metadata:
  Creation Timestamp:  2024-08-05T16:30:41Z
  Resource Version:    84174142
  UID:                 cffdba16-ff7a-4149-8c49-2c56d2be4946
Spec:
  Group:                     custom.metrics.k8s.io
  Group Priority Minimum:    100
  Insecure Skip TLS Verify:  true
  Service:
    Name:            prometheus-adapter
    Namespace:       monitoring
    Port:            443
  Version:           v1beta1
  Version Priority:  100
Status:
  Conditions:
    Last Transition Time:  2024-08-05T16:31:20Z
    Message:               all checks passed
    Reason:                Passed
    Status:                True
    Type:                  Available
Events:                    <none>

```

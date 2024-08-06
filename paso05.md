# PASO 5 DE CHALLENGE 04 : CREAR HPA USANDO LA MÉTRICA EXTERNA

## VERIFICAR API SERVICE

El API Service "v1beta1.custom.metrics.k8s.io" es importante porque es el intermediario que permitirá que el HPA pueda colectar la información de "prometheus-adapter".

Por ello verificamos que dicho servicio está disponible, como se ve a continuación:

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

## CREACIÓN DE HPA


Creamos el archivo "hpa.yaml" con la configuración que se muestra a continuación. Notar que el selector hace match a la app: bottelapp y el puerto "metrics" que fue definido previamente en el servicio que creamos en el paso 2

```
ubuntu@ubuntu:~/challenge-4/MYCHART$ cat <<EOF > templates/hpa.yaml
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: bottlehpa
  namespace: challenger-011
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: bottleapp
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Pods
    pods:
      metric:
        name: heavywork_per_second
      target:
        type: AverageValue
        averageValue: 10
EOF
```

Actualizamos la aplicación usando Helm para que incluye el "HPA"

```
ubuntu@ubuntu:~/challenge-4/MYCHART$ helm upgrade bottleapp . --namespace challenger-011
Release "bottleapp" has been upgraded. Happy Helming!
NAME: bottleapp
LAST DEPLOYED: Tue Aug  6 01:56:21 2024
NAMESPACE: challenger-011
STATUS: deployed
REVISION: 3
TEST SUITE: None
```



## VERIFICACIÓN




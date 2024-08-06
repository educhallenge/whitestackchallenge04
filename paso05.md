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

Vamos a crear un HPA que escale el deployment llamado "bottleapp" con un mínimo de 1 réplica y un máximo de 5 réplicas. Para realizar dicho autoscaling utilizamos la métrica "heavywork_per_second" creada por "prometheus-adapter" en el paso anterior.

Creamos el archivo "hpa.yaml" con la configuración que se muestra a continuación. 

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

A través de los logs de "prometheus-adapter" vemos que HPA hace la consulta de la métrica "heavywork_per_second" y la respuesta de "prometheus-adapter" es 200 lo cual significa que la respuesta está bien

```
ubuntu@ubuntu:~/challenge-4/MYCHART$ kubectl logs -l=app.kubernetes.io/name=prometheus-adapter -n monitoring -f | grep challenger-011
I0806 02:04:07.998973       1 httplog.go:132] "HTTP" verb="GET" URI="/apis/custom.metrics.k8s.io/v1beta1/namespaces/challenger-011/pods/%2A/heavywork_per_second?labelSelector=app%3Dbottleapp" latency="46.234896ms" userAgent="kube-controller-manager/v1.28.2+rke2r1 (linux/amd64) kubernetes/89a4ea3/system:serviceaccount:kube-system:horizontal-pod-autoscaler" audit-ID="eabbba07-8ba5-486c-8e57-8a0e26dd6f50" srcIP="10.42.5.192:53482" resp=200

I0806 02:04:23.032481       1 httplog.go:132] "HTTP" verb="GET" URI="/apis/custom.metrics.k8s.io/v1beta1/namespaces/challenger-011/pods/%2A/heavywork_per_second?labelSelector=app%3Dbottleapp" latency="18.595486ms" userAgent="kube-controller-manager/v1.28.2+rke2r1 (linux/amd64) kubernetes/89a4ea3/system:serviceaccount:kube-system:horizontal-pod-autoscaler" audit-ID="4fde6f60-a5db-4111-a352-7741a1476883" srcIP="10.42.5.192:53482" resp=200
```

También vemos que HPA ha levantado exitosamente y en este momento la métrica "heavywork_per_second" es 0 requests por segundo y que el target es 10 requests por segundo.

```
ubuntu@ubuntu:~/challenge-4/MYCHART$ kubectl get hpa
NAME        REFERENCE              TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
bottlehpa   Deployment/bottleapp   0/10      1         5         1          7m13s

ubuntu@ubuntu:~/challenge-4/MYCHART$ kubectl describe hpa bottlehpa
Name:                              bottlehpa
Namespace:                         challenger-011
Labels:                            app.kubernetes.io/managed-by=Helm
Annotations:                       meta.helm.sh/release-name: bottleapp
                                   meta.helm.sh/release-namespace: challenger-011
CreationTimestamp:                 Tue, 06 Aug 2024 02:00:07 +0000
Reference:                         Deployment/bottleapp
Metrics:                           ( current / target )
  "heavywork_per_second" on pods:  0 / 10
Min replicas:                      1
Max replicas:                      5
Deployment pods:                   1 current / 1 desired
Conditions:
  Type            Status  Reason            Message
  ----            ------  ------            -------
  AbleToScale     True    ReadyForNewScale  recommended size matches current size
  ScalingActive   True    ValidMetricFound  the HPA was able to successfully calculate a replica count from pods metric heavywork_per_second
  ScalingLimited  True    TooFewReplicas    the desired replica count is less than the minimum replica count
Events:           <none>
```

NOTA: también podríamos usar el comando `kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1` para verificar las métricas de "prometheus-adapter" pero no tenemos autorización para ver ese recurso


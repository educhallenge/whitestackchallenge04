# PASO 4 DE CHALLENGE 04 : INSTALAR Y CONFIGURAR PROMETHEUS ADAPTER

## CONFIGURAR Y VERIFICAR PROMETHEUS ADAPTER

- Pod y Servicio "prometheus-adapter"
Verificamos que el servicio "prometheus-adapter" está levantado por defecto en el namespace monitoring como se ve a continuación:

```
ubuntu@ubuntu:~$ kubectl get svc prometheus-adapter -n monitoring
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
prometheus-adapter   ClusterIP   10.43.184.6   <none>        443/TCP   45m
```

- Servicio "prometheus-operated"
Verificamos que el servicio "prometheus-operated" está en el namespace "monitoring" y usa el TCP 9090 como se ve a continuación:

```
ubuntu@ubuntu:~$ kubectl describe svc prometheus-operated -n monitoring
Name:              prometheus-operated
Namespace:         monitoring
Labels:            managed-by=prometheus-operator
                   operated-prometheus=true
Annotations:       <none>
Selector:          app.kubernetes.io/name=prometheus
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                None
IPs:               None
Port:              http-web  9090/TCP
TargetPort:        http-web/TCP
Endpoints:         10.42.107.35:9090
Session Affinity:  None
Events:            <none>
```

- URL de Prometheus para comunicación con "prometheus-adapter"

El formato de la URL de Prometheus tiene el formato http://prometheus-operated.namespace.svc:port  Con la información del comando anterior averiguamos que el URL de Prometheus es http://prometheus-operated.monitoring.svc:9090

Debemos verificar que la configuración del deployment de prometheus-adapter tiene dicho URL para comunicarse con Prometheus como se ve a continuación:

```
ubuntu@ubuntu:~$ kubectl describe deploy prometheus-adapter -n monitoring | grep url
      --prometheus-url=http://prometheus-operated.monitoring.svc:9090
```

- Configmap "prometheus-adapter" y rules

Además el mismo deployment de prometheus-adapter utiliza un configmap llamado también "prometheus-adapter" como se ve a continuación
```
ubuntu@ubuntu:~$ kubectl describe deploy prometheus-adapter -n monitoring | grep -A 4 Volumes
  Volumes:
   config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      prometheus-adapter
    Optional:  false
```



## VERIFICAR API SERVICE

Verificamos que el API Service v1beta1.custom.metrics.k8s.io se comunica correctamente con el servicio prometheus-adapter como se ve a continuación:
```
ubuntu@ubuntu:~$ kubectl get apiservices | grep "prometheus\|NAME"
NAME                                     SERVICE                         AVAILABLE   AGE
v1beta1.custom.metrics.k8s.io            monitoring/prometheus-adapter   True        48m

ubuntu@ubuntu:~$ kubectl describe apiservices v1beta1.custom.metrics.k8s.io
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
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

# PASO 4 DE CHALLENGE 04 : INSTALAR Y CONFIGURAR PROMETHEUS ADAPTER

## VERIFICAR DEPLOYMENT, PODS Y SERVICIOS NECESARIOS

- Pod y Servicio "prometheus-adapter"
  
Verificamos que el servicio "prometheus-adapter" está levantado por defecto en el namespace monitoring como se ve a continuación:

```
ubuntu@ubuntu:~$ kubectl get pod,svc -l  app.kubernetes.io/name=prometheus-adapter -n monitoring
NAME                                      READY   STATUS    RESTARTS   AGE
pod/prometheus-adapter-5776669bd7-6vltz   1/1     Running   0          6h5m

NAME                         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/prometheus-adapter   ClusterIP   10.43.184.6   <none>        443/TCP   8h
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

## CONFIGMAP Y RULES PARA "PROMETHEUS-ADAPTER"

- Verificar cuál es el configmap "prometheus-adapter"

El mismo deployment de prometheus-adapter nos dice que utiliza un configmap llamado también "prometheus-adapter" como se ve a continuación
```
ubuntu@ubuntu:~$ kubectl describe deploy prometheus-adapter -n monitoring | grep -A 4 Volumes
  Volumes:
   config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      prometheus-adapter
    Optional:  false
```

- Editamos el config map "prometheus-adapter" y pegamos la "rule" que vemos a continuación.

```
ubuntu@ubuntu:~$ kubectl edit cm prometheus-adapter -n monitoring
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  config.yaml: | 
rules:
- seriesQuery: 'heavywork_total{namespace!="",pod!=""}'
  resources:
    overrides:
      namespace:
        resource: namespace
      pod:
        resource: pod
  name:
    matches: "^(.*)_total"
    as: "${1}_per_second"
  metricsQuery: 'sum(rate(<<.Series>>{<<.LabelMatchers>>}[1m])) by (<<.GroupBy>>)'
#### RESTO DEL OUTPUT OMITIDO POR BREVEDAD
```

- Breve explicación de la "rule" que hemos pegado en el configmap

El "seriesQuery" recolecta las métricas del contador "heavywork_total" que obtiene de Prometheus.
```
- seriesQuery: 'heavywork_total{namespace!="",pod!=""}'
```

La sección "name" sirve para crear una métrica a partir del nombre "heavywork_total" le quita la palabra "total" y le agrega las palabras "per_second" creando así la métrica "heavywork_per_second". Dicha métrica tendrá el valor calculado por la "metricsQuery" que en este caso calcula el promedio de los valores recibidos en "heavywork_total".
```
name:
    matches: "^(.*)_total"
    as: "${1}_per_second"
  metricsQuery: 'sum(rate(<<.Series>>{<<.LabelMatchers>>}[1m])) by (<<.GroupBy>>)'
```


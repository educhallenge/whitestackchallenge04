# PASO 3 DE CHALLENGE 04 : CREAR SERVICE MONITOR PARA OBTENCIÓN DE MÉTRICAS

## CONFIGURACIÓN DEL SERVICE MONITOR

Creamos el archivo "servicemonitor.yaml" con la configuración que se muestra a continuación.Notar que el selector hace match a la app: bottelapp y el puerto "metrics" que fue definido previamente en el servicio que creamos en el paso 2

```
ubuntu@ubuntu:~/challenge-4/MYCHART$ cat <<EOF > templates/servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: bottleservicemonitor
  labels:
    app: bottleapp
    release: kube-prometheus-stack
spec:
  selector:
    matchLabels:
      app: bottleapp
  endpoints:
  -  port: metrics
     interval: 5s
EOF
```

Actualizamos la aplicación usando Helm para que incluye el "service monitor"

```
ubuntu@ubuntu:~/challenge-4/MYCHART$ helm upgrade bottleapp . --namespace challenger-011
Release "bottleapp" has been upgraded. Happy Helming!
NAME: bottleapp
LAST DEPLOYED: Mon Aug  5 00:46:21 2024
NAMESPACE: challenger-011
STATUS: deployed
REVISION: 2
TEST SUITE: None
```

## VERIFICACIÓN

- Verificamos con kubectl
```
ubuntu@ubuntu:~/challenge-4/MYCHART$ kubectl get servicemonitor -owide
NAME                   AGE
bottleservicemonitor   2m21s
```

- Hacemos port-forward para poder navegar a la web de Prometheus desde nuestra PC local
```
ubuntu@ubuntu:~/challenge-4/MYCHART$ kubectl port-forward service/kube-prometheus-stack-prometheus 9090 --address="0.0.0.0" -n monitoring
Forwarding from 0.0.0.0:9090 -> 9090
```

- Desde nuestra PC local con un navegador web podemos ver que Prometheus descubrió el target en nuestro namespace challenger-011 como se ve en la figura de abajo.

<img src="./prometheus_target.PNG">

- También podemos ver que Prometheus puede colectar la información "metrics" donde el contador "heavywork_total" subió a 2 como se muestra en la figura de abajo. Esto es debido a los 2 requests que se hicieron con curl a la ruta /heavywork en la verificación del paso 02.

<img src="./prometheus_graph_heavywork.PNG">

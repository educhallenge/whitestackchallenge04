PASO 3 DE CHALLENGE 04 : CREAR SERVICE MONITOR PARA OBTENCIÓN DE MÉTRICAS

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

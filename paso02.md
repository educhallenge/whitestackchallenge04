# PASO 2 DE CHALLENGE 04 : DESPLEGAR LA APLICACIÓN

## CREAR EL CONTAINER

Vamos al directorio "app" del repositorio que clonamos en el paso 1  y ejecutamos el siguiente comando para crear el container de manera local

```
ubuntu@ubuntu:~/challenge-4/ws-challenge-4/app$ ls -hal
total 20K
drwxrwxr-x 2 ubuntu ubuntu 4.0K Aug  1 23:37 .
drwxrwxr-x 4 ubuntu ubuntu 4.0K Aug  1 23:38 ..
-rw-rw-r-- 1 ubuntu ubuntu  664 Aug  1 23:34 app.py
-rw-rw-r-- 1 ubuntu ubuntu  450 Aug  1 06:06 Dockerfile
-rw-rw-r-- 1 ubuntu ubuntu   42 Aug  1 23:37 requirements.txt

ubuntu@ubuntu:~/challenge-4/ws-challenge-4/app$ sudo docker build -t edual/bottleapp:1.0 .
```

Verificamos que se creó el container localmente
```
ubuntu@ubuntu:~/challenge-4/ws-challenge-4/app$ sudo docker images
REPOSITORY        TAG       IMAGE ID       CREATED         SIZE
edual/bottleapp   1.0       a663ce61bcac   9 seconds ago   133MB
```

Hacemos login en Docker Hub
```
ubuntu@ubuntu:~/challenge-4/ws-challenge-4/app$ sudo docker login -u edual
Password:
Login Succeeded
```

Subimos el container al repositorio público Docker Hub
```
ubuntu@ubuntu:~/challenge-4/ws-challenge-4/app$ sudo docker image push edual/bottleapp:1.0
The push refers to repository [docker.io/edual/bottleapp]
beb69190ef0b: Pushed
8414a66ec11a: Pushed
e6259ac16ded: Pushed
ec6949936a52: Mounted from library/python
f71fab544a97: Mounted from library/python
d42276be00b5: Mounted from library/python
cc2286334a7b: Mounted from library/python
e0781bc8667f: Mounted from library/python
1.0: digest: sha256:1cf5aad63667dbaec6a3cd1c8747b860a7fda244ba321ca540649a169a6a2110 size: 1994
```

## HELM CHARTS

1. Creamos el directorio MYCHART y en dicho directorio creamos el archivo Chart.yaml con el contenido que vemos abajo

```
ubuntu@ubuntu:~/challenge-4$ mkdir MYCHART
ubuntu@ubuntu:~/challenge-4$ cd MYCHART
ubuntu@ubuntu:~/challenge-4/MYCHART$ cat <<EOF > Chart.yaml
apiVersion: v2
name: BOTTLE
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.16.0"
EOF
```
2. Creamos el subdirectorio "templates" y allí creamos el archivo "deployment.yaml" con el siguiente contenido. Notar que usamos el container "edual/bottleapp:1.0" que creamos en la sección anterior

```
ubuntu@ubuntu:~/challenge-4/MYCHART$ mkdir templates
ubuntu@ubuntu:~/challenge-4/MYCHART$ cat <<EOF > templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bottleapp
  labels:
    app: bottleapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bottleapp
  template:
    metadata:
      labels:
        app: bottleapp
    spec:
      containers:
      - name: bottleapp
        image: edual/bottleapp:1.0
        ports:
        - containerPort: 8080
EOF
```

3.  En el subdirectorio "templates" creamos el archivo "service.yaml" con el siguiente contenido. Notar que el servicio se llama ""bottleapp-service" y que damos el nombre "metrics" al puerto 8080 del servicio.  

```
ubuntu@ubuntu:~/challenge-4/MYCHART$ cat <<EOF > templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: bottleapp-service
  namespace: challenger-011
  labels:
    app: bottleapp
spec:
  selector:
    app: bottleapp
  ports:
    - name: metrics
      protocol: TCP
      port: 8080
      targetPort: 8080
EOF
```

4.  En el subdirectorio "templates" creamos el archivo "ingress.yaml" con el siguiente contenido. Notar que ingress está apuntando al servicio "bottleapp-service""

```
ubuntu@ubuntu:~/challenge-4/MYCHART$ cat <<EOF > templates/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bottle-ingress
  namespace: challenger-011
spec:
  rules:
  - host: mychallenge04.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: bottleapp-service
            port:
              number: 8080
EOF
```

5. Desplegamos el Chart para crear la aplicación
```
ubuntu@ubuntu:~/challenge-4/MYCHART$  helm install bottleapp . --namespace challenger-011
NAME: bottleapp
LAST DEPLOYED: Mon Aug  5 00:02:09 2024
NAMESPACE: challenger-011
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

## VERIFICACION CON HELM Y KUBECTL

Verificamos el despliegue con "helm ls"
```
ubuntu@ubuntu:~/challenge-4/MYCHART$ helm ls
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
bottleapp       challenger-011  1               2024-08-05 00:02:09.376585617 +0000 UTC deployed        BOTTLE-0.1.0    1.16.0
```

Verificamos que los objetos ingress, pod, service, deployment, replicaset hayan levantado correctmanete":
```
ubuntu@ubuntu:~/challenge-4/MYCHART$ kubectl get all,ingress
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME                             READY   STATUS    RESTARTS   AGE
pod/bottleapp-645fc9d6bb-fc9xj   1/1     Running   0          2m32s

NAME                        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/bottleapp-service   ClusterIP   10.43.97.67   <none>        8080/TCP   2m33s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/bottleapp   1/1     1            1           2m33s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/bottleapp-645fc9d6bb   1         1         1       2m33s

NAME                                       CLASS    HOSTS               ADDRESS         PORTS   AGE
ingress.networking.k8s.io/bottle-ingress   <none>   mychallenge04.com   10.43.114.145   80      2m34s
```

Verificamos que el ingress apunta el servicio bottleapp-service:8080 que a su vez apunta al ip del Pod 10.42.107.40:8080
```
ubuntu@ubuntu:~/challenge-4/MYCHART$ kubectl get pod -owide
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME                         READY   STATUS    RESTARTS   AGE     IP             NODE                                        NOMINATED NODE   READINESS GATES
bottleapp-645fc9d6bb-fc9xj   1/1     Running   0          8m10s   10.42.107.40   whitestackchallenge-worker-f97ebc81-gbspf   <none>           <none>

ubuntu@ubuntu:~/challenge-4/MYCHART$ kubectl describe ingress.networking.k8s.io/bottle-ingress
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
Name:             bottle-ingress
Labels:           app.kubernetes.io/managed-by=Helm
Namespace:        challenger-011
Address:          10.43.114.145
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host               Path  Backends
  ----               ----  --------
  mychallenge04.com
                     /   bottleapp-service:8080 (10.42.107.40:8080)
Annotations:         field.cattle.io/publicEndpoints:
                       [{"addresses":["10.43.114.145"],"port":80,"protocol":"HTTP","serviceName":"challenger-011:bottleapp-service","ingressName":"challenger-011...
                     meta.helm.sh/release-name: bottleapp
                     meta.helm.sh/release-namespace: challenger-011
Events:
  Type    Reason  Age                    From                      Message
  ----    ------  ----                   ----                      -------
  Normal  Sync    2m13s (x3 over 2m57s)  nginx-ingress-controller  Scheduled for sync
  Normal  Sync    2m13s (x3 over 2m57s)  nginx-ingress-controller  Scheduled for sync
  Normal  Sync    2m13s (x3 over 2m57s)  nginx-ingress-controller  Scheduled for sync
```

## VERIFICACION CON CURL

Temporalmente creamos un container con la aplicación cURL y luego ingresamos al shell de dicho container

```
ubuntu@ubuntu:~/challenge-4/MYCHART$ kubectl run mycurlpod --image=curlimages/curl -i --tty -- sh
```

Desde dicho shell podemos ejecutar cURL hacia el host "mychallenge04.com" con IP 10.43.114.145 que fueron definidos en el ingress. Vemos que la ruta /metric efectivamente está incrementando el contador cada vez que se hace un POST a la ruta /heavywork

```
~ $ curl -X POST http://10.43.114.145/heavywork -H 'Host: mychallenge04.com'
{"message": "Heavy work started"}~ $

~ $ curl -X POST http://10.43.114.145/lightwork -H 'Host: mychallenge04.com'
{"message": "Light work done"}~ $

~ $ curl -X POST http://10.43.114.145/lightwork -H 'Host: mychallenge04.com'
{"message": "Light work done"}~ $
~ $
~ $
~ $ curl -X POST http://10.43.114.145/heavywork -H 'Host: mychallenge04.com'
{"message": "Heavy work started"}~ $
~ $
~ $ curl http://10.43.114.145/metrics -H 'Host: mychallenge04.com'
# HELP heavywork_total heavywork metric
# TYPE heavywork_total counter
heavywork_total 2.0
# HELP heavywork_created heavywork metric
# TYPE heavywork_created gauge
heavywork_created 1.7228299355643427e+09
```

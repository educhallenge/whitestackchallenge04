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

Publicamos el container en el repositorio público Docker Hub
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




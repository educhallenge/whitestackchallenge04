# PASO 0 DE CHALLENGE 04 : PROBAR ACCESO

## CONFIGURACION

Copiamos el archivo "whitestackchallenge.yaml" a nuestra máquina local Linux y luego creamos una variable de entorno KUBECONFIG que será usada para conectarnos al entorno remoto de Kubernetes

```
ubuntu@ubuntu:~$ echo "export KUBECONFIG=~/challenge-4/whitestackchallenge.yaml" >> ~/.profile
```

## VERIFICACION

Probamos el acceso remoto listando los pods de Kubernetes remoto

```
ubuntu@ubuntu:~$ kubectl get pods -n challenger-011
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
No resources found in challenger-011 namespace.
```

```
ubuntu@ubuntu:~$ kubectl get cm -n monitoring
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME                                                      DATA   AGE
kube-root-ca.crt                                          1      76d
prometheus-adapter-config                                 1      2d12h
prometheus-kube-prometheus-stack-prometheus-rulefiles-0   35     76d
```

```
ubuntu@ubuntu:~$ kubectl get all -n monitoring
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME                                                            READY   STATUS    RESTARTS   AGE
pod/kube-prometheus-stack-kube-state-metrics-65594f9476-d55w5   1/1     Running   0          76d
pod/kube-prometheus-stack-operator-6459f9c556-mrgk6             1/1     Running   0          76d
pod/kube-prometheus-stack-prometheus-node-exporter-2djhm        1/1     Running   0          76d
pod/kube-prometheus-stack-prometheus-node-exporter-9kftf        1/1     Running   0          76d
pod/kube-prometheus-stack-prometheus-node-exporter-vmnld        1/1     Running   0          76d
pod/prometheus-kube-prometheus-stack-prometheus-0               2/2     Running   0          76d

NAME                                                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/kube-prometheus-stack-kube-state-metrics         ClusterIP   10.43.190.67    <none>        8080/TCP            76d
service/kube-prometheus-stack-operator                   ClusterIP   10.43.163.141   <none>        443/TCP             76d
service/kube-prometheus-stack-prometheus                 ClusterIP   10.43.226.247   <none>        9090/TCP,8080/TCP   76d
service/kube-prometheus-stack-prometheus-node-exporter   ClusterIP   10.43.47.97     <none>        9100/TCP            76d
service/prometheus-adapter                               ClusterIP   10.43.57.97     <none>        443/TCP             11d
service/prometheus-operated                              ClusterIP   None            <none>        9090/TCP            76d

NAME                                                            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-prometheus-stack-prometheus-node-exporter   3         3         3       3            3           kubernetes.io/os=linux   76d

NAME                                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kube-prometheus-stack-kube-state-metrics   1/1     1            1           76d
deployment.apps/kube-prometheus-stack-operator             1/1     1            1           76d

NAME                                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/kube-prometheus-stack-kube-state-metrics-65594f9476   1         1         1       76d
replicaset.apps/kube-prometheus-stack-operator-6459f9c556             1         1         1       76d

NAME                                                           READY   AGE
statefulset.apps/prometheus-kube-prometheus-stack-prometheus   1/1     76d
```

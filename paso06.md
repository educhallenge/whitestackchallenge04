# PASO 6 DE CHALLENGE 04 : GENERAR CARGA Y ANALIZAR

En nuestro Linux local hacemos un port-forward para poder enviar requests al servicio que creamos en el namespace challenger-011

```
$ kubectl port-forward service/bottleapp-service 8080
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080

```

Modificamos el archivo generate_load.py para soportar múltiples hilos y así generar mayor carga. En nuestro Linux local abrimos otra terminal para ejecutar el archivo con python y enviar el http request a la ruta /heavywork

```
$ python3 generate_load.py 127.0.0.1 8080 /heavywork
202 {"message": "Heavy work started"}
202 {"message": "Heavy work started"}
202 {"message": "Heavy work started"}
202 {"message": "Heavy work started"}

```

En nuestro Linux local abrimos una terminal más para monitorear el status de HPA. Notamos que cuando excede el target de 10 se despliega automáticamente 1 pod más

```
ubuntu@ubuntu:~$ kubectl describe hpa bottlehpa
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
Name:                              bottlehpa
Namespace:                         challenger-011
Labels:                            app.kubernetes.io/managed-by=Helm
Annotations:                       meta.helm.sh/release-name: bottleapp
                                   meta.helm.sh/release-namespace: challenger-011
CreationTimestamp:                 Tue, 06 Aug 2024 02:00:07 +0000
Reference:                         Deployment/bottleapp
Metrics:                           ( current / target )
  "heavywork_per_second" on pods:  11927m / 10
Min replicas:                      1
Max replicas:                      5
Deployment pods:                   1 current / 2 desired
Conditions:
  Type            Status  Reason              Message
  ----            ------  ------              -------
  AbleToScale     True    SucceededRescale    the HPA controller was able to update the target scale to 2
  ScalingActive   True    ValidMetricFound    the HPA was able to successfully calculate a replica count from pods metric heavywork_per_second
  ScalingLimited  False   DesiredWithinRange  the desired count is within the acceptable range
Events:
  Type    Reason             Age   From                       Message
  ----    ------             ----  ----                       -------
  Normal  SuccessfulRescale  8s    horizontal-pod-autoscaler  New size: 2; reason: pods metric heavywork_per_second above target
```
```
ubuntu@ubuntu:~$ kubectl get deploy
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
bottleapp   2/2     2            2           25h
```
```
ubuntu@ubuntu:~$ kubectl get pod
NAME                         READY   STATUS    RESTARTS     AGE
bottleapp-645fc9d6bb-2w2nm   1/1     Running   0            75s
bottleapp-645fc9d6bb-fc9xj   1/1     Running   0            25h
```

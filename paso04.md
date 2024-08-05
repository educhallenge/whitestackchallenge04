# PASO 4 DE CHALLENGE 04 : INSTALAR Y CONFIGURAR PROMETHEUS ADAPTER

Verificamos la existencia del configmap de Prometheus

```
ubuntu@ubuntu:~/challenge-4/MYCHART$ kubectl get cm -n monitoring
Warning: Use tokens from the TokenRequest API or manually created secret-based tokens instead of auto-generated secret-based tokens.
NAME                                                      DATA   AGE
kube-root-ca.crt                                          1      78d
prometheus-kube-prometheus-stack-prometheus-rulefiles-0   35     78d

```

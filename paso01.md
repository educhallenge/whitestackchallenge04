# PASO 1 DE CHALLENGE 04 : ADAPTAR LA APLICACION WEB

Clonamos la aplicación desde el repositorio que nos compartieron

```
git clone https://github.com/whitestack/ws-challenge-4.git
```

Editamos el archivo requirements para que incluya también a prometheus_client como se ve abajo

```
ubuntu@ubuntu:~/challenge-4/ws-challenge-4/app$ more requirements.txt
bottle==0.12.19
prometheus_client==0.20.0
```

En el archivo "app.py" agregamos las siguientes líneas para importar prometheus_client y crear el counter llamado "heavywork_metric"

```
from prometheus_client import Counter, generate_latest
heavywork_metric = Counter('heavywork', 'heavywork metric')
```

En la función heavywork() incluimos una línea que incrementará el contador "heavywork_metric" cada vez que se llame al método POST de la ruta /heavywork

```
@app.post('/heavywork')
def heavywork():
    response.status = 202
    heavywork_metric.inc()
    return {"message": "Heavy work started"}
```

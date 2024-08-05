# PASO 1 DE CHALLENGE 04 : ADAPTAR LA APLICACION WEB

## CONFIGURACION

1. Clonamos la aplicación desde el repositorio que nos compartieron

```
git clone https://github.com/whitestack/ws-challenge-4.git
```

2. Editamos el archivo requirements para que incluya también a prometheus_client como se ve abajo

```
ubuntu@ubuntu:~/challenge-4/ws-challenge-4/app$ more requirements.txt
bottle==0.12.19
prometheus_client==0.20.0
```

3. En el archivo "app.py" agregamos las siguientes líneas para importar prometheus_client y crear el counter llamado "heavywork_metric"

```
from prometheus_client import Counter, generate_latest
heavywork_metric = Counter('heavywork', 'heavywork metric')
```

Incluimos la línea "heavywork_metric.inc()" que incrementará el contador "heavywork_metric" cada vez que se llame al método POST de la ruta /heavywork

```
@app.post('/heavywork')
def heavywork():
    response.status = 202
    heavywork_metric.inc()
    return {"message": "Heavy work started"}
```

Creamos la ruta /metrics para que devuelva el contenido de la métrica heavywork_metric en texto plano

```
@app.route('/metrics')
def metrics():
    response.content_type = 'text/plain; charset=utf-8'
    return generate_latest(heavywork_metric)
```

Después de hacer las modificaciones mencionadas arriba el archivo "app.py" queda de la siguiente forma:

```
ubuntu@ubuntu:~/challenge-4/ws-challenge-4/app$ more app.py
from bottle import Bottle, request, response
from prometheus_client import Counter, generate_latest

heavywork_metric = Counter('heavywork', 'heavywork metric')

app = Bottle()

@app.post('/heavywork')
def heavywork():
    response.status = 202
    heavywork_metric.inc()
    return {"message": "Heavy work started"}


@app.post('/lightwork')
def lightwork():
    return {"message": "Light work done"}

@app.route('/metrics')
def metrics():
    response.content_type = 'text/plain; charset=utf-8'
    return generate_latest(heavywork_metric)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

## VERIFICACION

1. Ejecutamos la aplicación
```
ubuntu@ubuntu:~/challenge-4/ws-challenge-4/app$ python3 app.py
Bottle v0.12.25 server starting up (using WSGIRefServer())...
Listening on http://0.0.0.0:8080/
Hit Ctrl-C to quit.
```

2. Desde una PC Windows hacemos las consultas HTTP con el comando CURL y notamos que el contador heavywork_total se incrementa cada vez que se llama al método POST de "192.168.0.116:8080/heavywork"

```
C:\>curl 192.168.0.116:8080/metrics
# HELP heavywork_total heavywork metric
# TYPE heavywork_total counter
heavywork_total 0.0
# HELP heavywork_created heavywork metric
# TYPE heavywork_created gauge
heavywork_created 1.722670485619919e+09


C:\>curl -X POST 192.168.0.116:8080/heavywork
{"message": "Heavy work started"}

C:\>curl -X POST 192.168.0.116:8080/heavywork
{"message": "Heavy work started"}

C:\>curl -X POST 192.168.0.116:8080/lightwork
{"message": "Light work done"}

C:\>curl 192.168.0.116:8080/metrics
# HELP heavywork_total heavywork metric
# TYPE heavywork_total counter
heavywork_total 2.0
# HELP heavywork_created heavywork metric
# TYPE heavywork_created gauge
heavywork_created 1.722670485619919e+09
```

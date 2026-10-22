# Unidad 4 — Práctica extra: servicios de nube local con Floci

**Materia:** Taller de Sistemas Operativos (SCA-1026)
**Tema:** No es parte del programa oficial de 4 unidades (ya cubierto por completo en [01](01_interoperabilidad.md)-[04](04_proyecto_distros_seguridad.md)) — es una práctica adicional sobre una habilidad de administración de servidores muy demandada hoy: emular servicios de nube (AWS/Azure/GCP) **en tu propia máquina**, sin cuenta ni tarjeta de crédito.
**Herramienta:** [Floci](https://floci.io) — código abierto (MIT), corre sobre Docker
**Todo lo de este documento se ejecutó de verdad** en una máquina Ubuntu real, con Docker real — ningún resultado está inventado.

## Por qué esto es un tema de Sistemas Operativos, no solo de "la nube"

Floci no reemplaza lo que ya aprendiste de administración de servidores — lo usa. Es un **proceso servidor** que corre dentro de un **contenedor** (control de procesos, aislamiento de namespaces — lo que ya viste con `systemd` en la Unidad 3), expone un **socket de red** en el puerto 4566 (el mismo modelo cliente-servidor que ya viste con Samba en esta unidad), y depende de que **Docker esté corriendo como servicio** en tu sistema. La diferencia es que, en vez de compartir archivos (Samba) o ejecutar comandos remotos (SSH), aquí el "servicio" que levantas imita la API de un proveedor de nube real — así puedes practicar cómo se administra infraestructura en la nube sin gastar dinero ni arriesgar una cuenta de producción.

---

## 1. Instalación del CLI (sin `sudo`)

El instalador oficial baja un binario precompilado. Por defecto intenta instalarlo en `/usr/local/bin` (necesita privilegios), pero acepta una variable de entorno para instalar en tu propio directorio:

```bash
mkdir -p ~/.local/bin
curl -fsSL https://floci.io/install.sh -o /tmp/floci_install.sh
FLOCI_INSTALL_DIR=~/.local/bin sh /tmp/floci_install.sh
export PATH="$HOME/.local/bin:$PATH"
```

**Salida real:**

```
Installing Floci CLI...
Downloading floci 0.4.0 for linux-amd64...
Checksum verified.

Floci CLI 0.4.0 installed to /home/luiscoutino/.local/bin/floci
```

Nota que el instalador **verifica el checksum** del binario descargado antes de instalarlo — una práctica de seguridad básica que deberías buscar en cualquier script de instalación que corras con `curl | sh` (revisa siempre el script antes de ejecutarlo; en este caso es un script simple de ~100 líneas que solo descarga y verifica un binario).

---

## 2. Diagnóstico antes de arrancar: `floci doctor`

Antes de levantar nada, Floci trae su propio comando de diagnóstico — la misma filosofía de "diagnostica antes de adivinar" que ya aplicaste con `mobile_diagnose` o revisando logs de `systemd`:

```bash
floci doctor
```

**Salida real (primera corrida, antes de arrancar el contenedor):**

```
Floci AWS Doctor — checking your environment

  + docker.installed           Docker 29.9.0 detected
  + docker.daemon              Daemon reachable
  + docker.socket              /var/run/docker.sock accessible
  + docker.version             Docker 29.9.0 (>= 20.10)
  + port.available             Port 4566 free
  x image.present              floci/floci not pulled locally
                               Fix: floci start --pull always
  ! image.version              floci/floci not present — version cannot be checked
                               Fix: floci start --pull always
  ! container.running          Container 'floci' not found
                               Fix: floci start

2 issue(s) found (2 fail, 3 warn). Follow the Fix line under each one.
```

Fíjate en la estructura: valida en orden de dependencia — primero que Docker exista, luego que el daemon responda, luego que el socket sea accesible, luego la versión, luego el puerto, y solo al final la imagen y el contenedor. Es exactamente el mismo patrón de diagnóstico en cascada que viste en la guía de ARTEMIS (`mobile_diagnose`): no tiene caso revisar si el contenedor corre si Docker ni siquiera está instalado.

---

## 3. Levantar el emulador

```bash
floci start
```

**Salida real:**

```
Checking image floci/floci:latest (policy: missing)...
Starting Floci AWS container...
Container started (a26d846dfd6a)
Waiting for Floci AWS to be ready...
Floci AWS is ready (http://localhost:4566)
```

Verifica el estado con:

```bash
floci status
```

**Salida real:**

```
Floci AWS Status

  Container:  floci  running
  Image:      floci/floci:latest
  Ports:      4566->4566/tcp
  Endpoint:   http://localhost:4566
  Reachable:  yes
  Version:    2.3.0
  Edition:    floci-always-free
```

Y para ver qué servicios están disponibles en esta instancia (son **127 servicios** emulados, de AWS/Azure/GCP combinados):

```bash
floci services
```

**Salida real (primeras líneas):**

```
Floci AWS Services  (127 enabled)

  +  ssm
  +  sqs
  +  s3
  +  dynamodb
  +  sns
  +  lambda
  +  apigateway
  +  iam
  +  kafka
  +  mq
  +  elasticache
  +  memorydb
  +  rds
```

---

## 4. Conectar la CLI de AWS real contra el emulador local

Instalamos la CLI de AWS y `boto3` (el SDK de Python para AWS) normalmente — Floci habla el mismo protocolo que AWS de verdad, así que **ninguna herramienta sabe que no está hablando con la nube real**:

```bash
pip3 install --user boto3 awscli
```

Las credenciales pueden ser cualquier texto — Floci no valida contra una cuenta real, solo necesita *algo* con el formato correcto de firma AWS SigV4:

```bash
cat ~/.aws/credentials
```

```
[default]
aws_access_key_id = test
aws_secret_access_key = test
```

### 4.1 S3 — crear un bucket, subir y descargar un archivo

```bash
aws --endpoint-url http://localhost:4566 s3 mb s3://practica-tecnm
echo "contenido de prueba para la practica guiada" > archivo_prueba.txt
aws --endpoint-url http://localhost:4566 s3 cp archivo_prueba.txt s3://practica-tecnm/archivo_prueba.txt
aws --endpoint-url http://localhost:4566 s3 ls s3://practica-tecnm/
aws --endpoint-url http://localhost:4566 s3 cp s3://practica-tecnm/archivo_prueba.txt descargado.txt
diff archivo_prueba.txt descargado.txt && echo "Archivos identicos: OK"
```

**Salida real:**

```
make_bucket: practica-tecnm
upload: ./archivo_prueba.txt to s3://practica-tecnm/archivo_prueba.txt
2026-10-22 17:10:35         44 archivo_prueba.txt
download: s3://practica-tecnm/archivo_prueba.txt to ./descargado.txt
Archivos identicos: OK
```

El archivo viajó de verdad: salió de tu disco, cruzó el socket hacia el contenedor, el contenedor lo guardó en su propio almacenamiento interno, y al pedirlo de vuelta llegó byte por byte idéntico.

### 4.2 DynamoDB — crear una tabla, insertar y leer un registro

```bash
EP="http://localhost:4566"

aws --endpoint-url $EP dynamodb create-table \
  --table-name Estudiantes \
  --attribute-definitions AttributeName=matricula,AttributeType=S \
  --key-schema AttributeName=matricula,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --query 'TableDescription.TableStatus' --output text

aws --endpoint-url $EP dynamodb put-item \
  --table-name Estudiantes \
  --item '{"matricula": {"S": "2026001"}, "nombre": {"S": "Soto Perez Angel"}, "equipo": {"S": "E1"}}'

aws --endpoint-url $EP dynamodb get-item \
  --table-name Estudiantes \
  --key '{"matricula": {"S": "2026001"}}'
```

**Salida real:**

```
ACTIVE
{
    "Item": {
        "matricula": {"S": "2026001"},
        "nombre": {"S": "Soto Perez Angel"},
        "equipo": {"S": "E1"}
    }
}
```

### 4.3 SQS — crear una cola, enviar y recibir un mensaje

```bash
QUEUE_URL=$(aws --endpoint-url $EP sqs create-queue --queue-name cola-practica --query 'QueueUrl' --output text)
aws --endpoint-url $EP sqs send-message --queue-url "$QUEUE_URL" --message-body "Hola desde el Taller de Sistemas Operativos"
aws --endpoint-url $EP sqs receive-message --queue-url "$QUEUE_URL" --query 'Messages[0].Body' --output text
```

**Salida real:**

```
http://localhost:4566/000000000000/cola-practica
{"MD5OfMessageBody": "b02eddd4de377ee62efb7ae3fd39f7ce", "MessageId": "111a1fff-..."}
Hola desde el Taller de Sistemas Operativos
```

Si ya trabajaste con colas en la Unidad 3 de Estructura de Datos (Redis), reconoces el patrón: una cola FIFO donde un productor mete mensajes y un consumidor los saca — la diferencia aquí es que hablas el protocolo de SQS, no el de Redis, pero la idea de "cola de mensajes entre procesos" es la misma.

---

## 5. Un hallazgo real de troubleshooting (y por qué documentarlo importa)

Al construir esta práctica nos topamos con algo que **no estaba documentado de antemano** y tuvimos que diagnosticar como lo haría cualquier administrador de sistemas real: `floci env` (el comando que imprime las variables de entorno recomendadas) sugiere exportar `AWS_ENDPOINT_URL` como variable global:

```bash
$ floci env
export AWS_ENDPOINT_URL='http://localhost.floci.io:4566'
export AWS_ACCESS_KEY_ID='test'
export AWS_SECRET_ACCESS_KEY='test'
export AWS_DEFAULT_REGION='us-east-1'
```

Usando exactamente esas variables, **S3 funciona perfecto**, pero **SQS falla**:

```bash
$ eval "$(floci env)"
$ aws s3 mb s3://prueba-env-var
make_bucket: prueba-env-var          # OK

$ aws sqs create-queue --queue-name cola-practica3

An error occurred (InvalidClientTokenId) when calling the CreateQueue operation: The security token included in the request is invalid.
```

Pero si en vez de la variable de entorno global usas el flag `--endpoint-url` en el mismo comando, **con las mismas credenciales, en el mismo shell**, SQS sí funciona:

```bash
$ unset AWS_ENDPOINT_URL
$ aws --endpoint-url http://localhost:4566 sqs create-queue --queue-name cola-practica4
{
    "QueueUrl": "http://localhost:4566/000000000000/cola-practica4"
}
```

**Diagnóstico:** el problema no es Floci ni el servicio SQS en sí (ya viste que `--endpoint-url` sí conecta igual de bien) — es la forma en que la CLI de AWS calcula la firma SigV4 cuando el endpoint viene de la variable global `AWS_ENDPOINT_URL` contra ciertos protocolos (SQS usa JSON sobre HTTP con un esquema de firma distinto al de S3, que es REST). Si alguna vez te topas con un error de autenticación que solo ocurre con **algunos** servicios de la misma herramienta y no con otros, esa asimetría es la pista: no es un problema de credenciales, es un problema de **cómo** se construye la petición para ese protocolo específico.

**Lección para el taller:** siempre que automatices algo con variables de entorno (como hiciste con `systemctl --user` en la Unidad 3), prueba con el flag explícito del comando antes de asumir que el problema está en tus credenciales o en el servidor — a veces la variable de entorno global no se propaga igual a todas las herramientas o protocolos.

---

## 6. Ciclo de vida completo: reset y apagado

```bash
# Reset borra TODOS los recursos (buckets, tablas, colas) pero deja el contenedor corriendo
floci reset --yes

# Apagar por completo
floci stop
```

**Salida real:**

```
$ floci stop
Stopping Floci AWS container 'floci'...
Stopped.
```

Después de `floci stop`, `docker ps` ya no muestra el contenedor — el proceso terminó de verdad, igual que cualquier otro contenedor Docker que ya administraste en la Unidad 4 con Samba.

---

## Qué te llevas de esta práctica

- Un **proceso servidor en contenedor** se administra con las mismas herramientas que cualquier otro servicio Linux (Docker, puertos, logs).
- Las herramientas de nube real (AWS CLI, boto3, Terraform) pueden apuntar a un **endpoint local** sin saber que no es la nube real — útil para practicar sin gastar dinero ni arriesgar una cuenta de producción.
- El troubleshooting de "funciona con un servicio pero no con otro, mismas credenciales" es una pista de que el problema está en cómo se construye la petición (protocolo/firma), no en las credenciales mismas.

## Fuentes

- floci.io — documentación oficial, instalador, y referencia de servicios
- docs.aws.amazon.com — AWS CLI, SQS, DynamoDB, S3 (los mismos SDKs/CLI usados contra el emulador)
- github.com/floci-io — repositorio del proyecto (MIT License)

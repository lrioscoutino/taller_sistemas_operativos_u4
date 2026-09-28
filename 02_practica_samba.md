# Práctica guiada: interoperabilidad con Samba

Práctica complementaria a [4.1 Interoperabilidad](01_interoperabilidad.md). Vas a levantar un servidor Samba real en Linux, compartir una carpeta, y conectarte a ella desde un cliente — la misma operación que haría un usuario de Windows al abrir `\\servidor\carpeta` en el explorador de archivos.

> Verificado en este sandbox con un servidor Samba real corriendo en Docker (imagen `dperson/samba`, Samba 4.12) y un cliente Python (`smbprotocol`) que habla el protocolo SMB de verdad — no es una simulación. Los comandos de instalación nativa (`apt install samba`) siguen la documentación oficial de Ubuntu/Debian.

## Requisitos previos

- Docker instalado (ver la práctica de Docker de esta materia, si no lo tienes).
- Python 3 con `uv` o `pip` (para el cliente de prueba).

## Paso 1 — Instalación nativa de Samba (para referencia)

En un servidor Linux real, sin Docker, así se instala:

```bash
sudo apt update
sudo apt install -y samba samba-common-bin smbclient cifs-utils

# Crear un usuario de sistema y darle contraseña de Samba (Samba mantiene su propia base de usuarios)
sudo useradd -M -s /usr/sbin/nologin alice     # -M: sin carpeta home, no necesita shell
sudo smbpasswd -a alice                         # pide una contraseña — esta es la de Samba, no la del SO

# Configurar el recurso compartido en /etc/samba/smb.conf
sudo tee -a /etc/samba/smb.conf << 'EOF'

[publico]
   path = /srv/samba/publico
   valid users = alice
   read only = no
   browsable = yes
EOF

sudo mkdir -p /srv/samba/publico
sudo chown alice:alice /srv/samba/publico

sudo testparm                    # valida la sintaxis de smb.conf antes de reiniciar
sudo systemctl restart smbd nmbd
```

## Paso 2 — Levantar el servidor (esta práctica, en Docker)

Para practicar sin necesitar una segunda máquina física, usa un contenedor con Samba real — mismo protocolo, mismo binario `smbd` por dentro:

```bash
mkdir -p ~/practica_samba/compartido
echo "Este archivo vive en el servidor Linux" > ~/practica_samba/compartido/bienvenida.txt

docker run -d --name samba-practica \
  -p 1445:445 \
  -v ~/practica_samba/compartido:/share \
  dperson/samba -p \
  -u "alice;alice123" \
  -s "publico;/share;yes;no;no;alice"

sleep 3
docker logs samba-practica
```

**Checkpoint esperado:** el log debe terminar con `smbd version ... started` y `daemon_ready`.

> Se usa el puerto `1445` en vez del `445` estándar porque en un sandbox el 445 puede estar ocupado o restringido — en un servidor real, `-p 445:445` es lo normal.

## Paso 3 — Ver la configuración activa del servidor

```bash
docker exec samba-practica testparm -s
```

**Checkpoint esperado:** debe mostrar `Server role: ROLE_STANDALONE`, el recurso `[publico]`, y protocolos `SMB2`/`SMB3` habilitados — la misma herramienta `testparm` que usarías en una instalación nativa para validar `/etc/samba/smb.conf` antes de reiniciar el servicio.

## Paso 4 — Conectarse como cliente y verificar interoperabilidad real

```bash
python3 -m venv ~/practica_samba/venv
~/practica_samba/venv/bin/pip install smbprotocol
```

```python
# guarda esto como cliente_samba.py y corre con: ~/practica_samba/venv/bin/python cliente_samba.py
import smbclient

kw = dict(username="alice", password="alice123", port=1445)
smbclient.register_session("127.0.0.1", **kw)

print("--- listar contenido del recurso compartido ---")
for entrada in smbclient.listdir(r"\\127.0.0.1\publico", **kw):
    print(entrada)

print("--- leer el archivo que ya existía en el servidor ---")
with smbclient.open_file(r"\\127.0.0.1\publico\bienvenida.txt", mode="r", **kw) as f:
    print(f.read())

print("--- escribir un archivo NUEVO desde el cliente ---")
with smbclient.open_file(r"\\127.0.0.1\publico\desde_cliente.txt", mode="w", **kw) as f:
    f.write("Este archivo lo creó el cliente por SMB\n")

print("--- confirmar que ya aparece en el listado ---")
for entrada in smbclient.listdir(r"\\127.0.0.1\publico", **kw):
    print(entrada)
```

**Checkpoint esperado:**
```
--- listar contenido del recurso compartido ---
bienvenida.txt
--- leer el archivo que ya existía en el servidor ---
Este archivo vive en el servidor Linux

--- escribir un archivo NUEVO desde el cliente ---
--- confirmar que ya aparece en el listado ---
bienvenida.txt
desde_cliente.txt
```

## Paso 5 — Confirmar que el archivo llegó al disco real del servidor

```bash
cat ~/practica_samba/compartido/desde_cliente.txt
ls -la ~/practica_samba/compartido/
```

**Checkpoint esperado:** el archivo `desde_cliente.txt` existe físicamente en la carpeta del servidor — no fue solo una respuesta simulada del cliente, el protocolo SMB de verdad escribió el archivo del otro lado de la red.

## Paso 6 — El mismo recurso, desde un cliente Windows real (para tu casa/laboratorio)

Si tienes acceso a una máquina Windows en la misma red que tu servidor Samba:

1. Abre el Explorador de archivos.
2. En la barra de direcciones escribe `\\<IP-del-servidor-Linux>\publico`.
3. Cuando pida credenciales, usa el usuario y contraseña de Samba (`alice` / `alice123` en esta práctica, no el usuario de Windows).
4. Debes poder ver, abrir y crear archivos en esa carpeta exactamente como si fuera una carpeta compartida de otra PC con Windows.

```bash
# comando equivalente desde otra máquina Linux, montando el recurso:
sudo mount -t cifs //<IP-del-servidor>/publico /mnt/samba -o username=alice,password=alice123
```

## Paso 7 — Ver las conexiones activas desde el servidor (administración real)

```bash
docker exec samba-practica smbstatus
```

Muestra las sesiones conectadas, qué archivos tienen abiertos, y desde qué IP — la misma información que un administrador de red revisaría en un servidor de archivos real para saber quién está usando qué recurso.

## Paso 8 — Limpieza

```bash
docker stop samba-practica
docker rm samba-practica
rm -rf ~/practica_samba
```

---

## Qué acabas de comprobar, en términos de la teoría (4.1)

| Lo que hiciste | Concepto que confirma |
|---|---|
| El servidor Linux compartió una carpeta con `smbd` | Samba implementa el protocolo SMB nativo de Windows, sobre Linux — interoperabilidad real de sistemas de archivos (4.1.1). |
| El cliente Python se conectó por el puerto 445/1445 | SMB corre sobre un socket TCP — la comunicación entre procesos de bajo nivel que vimos en 4.1.2. |
| `listdir`/`open_file` funcionaron como llamadas locales, aunque el archivo está en otra "máquina" (el contenedor) | Es exactamente la idea de RPC: llamar a una operación remota (leer/escribir un archivo) con la misma sintaxis que si fuera local. |
| `testparm` y `smbstatus` | Las mismas herramientas de administración que usarías en un servidor de archivos empresarial real. |

## Actividades de aprendizaje

- Repite el Paso 2 pero agregando un segundo usuario (`bob`) con acceso de **solo lectura** al mismo recurso (`valid users = alice, bob` + `write list = alice`), y verifica desde el cliente que `bob` puede leer pero no escribir.
- Investiga la diferencia entre un recurso `public` (invitado, sin contraseña) y uno con `valid users` como el de esta práctica — ¿cuándo usarías cada uno en una red real?
- Compara Samba (SMB) contra NFS: monta el mismo tipo de recurso compartido usando NFS entre dos contenedores Linux, y documenta qué comandos cambian y cuáles son conceptualmente idénticos.
- Investiga qué es Active Directory y cómo Samba puede actuar como controlador de dominio (`samba-tool domain provision`) — la pieza que permite integrarse con la administración de usuarios de Windows Server que viste en la Unidad 2.

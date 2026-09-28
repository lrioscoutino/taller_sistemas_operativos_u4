# 4.1 Interoperabilidad entre sistemas operativos

## Qué es y por qué existe

**Interoperabilidad** es la capacidad de dos sistemas operativos distintos (ej. Windows y Linux) para compartir recursos y comunicarse, aunque cada uno tenga su propio sistema de archivos, protocolos y forma de administrar usuarios. En cualquier red real coexisten estaciones Windows, servidores Linux, impresoras, y aplicaciones que necesitan hablar entre sí — la interoperabilidad es lo que evita que cada SO viva en una isla aparte.

## 4.1.1 Sistemas de archivos y recursos (NFS, impresoras)

| Mecanismo | Para qué sirve | Nativo de |
|---|---|---|
| **NFS** (Network File System) | Compartir carpetas entre sistemas Unix/Linux | Unix/Linux |
| **SMB/CIFS** (vía Samba en Linux) | Compartir carpetas e impresoras con Windows | Windows (Samba lo implementa en Linux) |
| **CUPS + IPP** | Compartir impresoras en red, multiplataforma | Linux/Unix (soportado también por Windows/macOS) |

La pieza central de esta unidad es **Samba**: un servidor de código abierto que corre en Linux y habla el protocolo **SMB** (Server Message Block) — el mismo que usa Windows para "Compartir carpetas e impresoras" nativamente. Gracias a Samba, un servidor Linux puede aparecer en el explorador de archivos de Windows exactamente como si fuera "otro Windows" en la red, sin que el usuario de Windows note ninguna diferencia.

```
┌─────────────┐     protocolo SMB/CIFS     ┌─────────────┐
│  Windows 10   │ ◄────────────────────────► │  Servidor    │
│  (cliente)    │      (puerto 445/139)       │  Linux+Samba │
└─────────────┘                              └─────────────┘
```

## 4.1.2 Comunicación entre procesos (Sockets, RPC)

Por debajo de NFS y SMB hay dos mecanismos genéricos de comunicación entre procesos que ya tocaste en otras materias:

- **Sockets** — la base de toda comunicación en red: un extremo abre un socket, el otro se conecta, y ambos intercambian bytes. SMB, HTTP, SSH — todos corren, en el fondo, sobre sockets TCP.
- **RPC** (Remote Procedure Call) — permite que un programa llame a una función que en realidad se ejecuta en otra máquina, como si fuera local. NFS usa RPC internamente (a través de `rpcbind`) para que el cliente "llame" a operaciones del servidor remoto (leer un archivo, listar un directorio) de forma transparente.

La diferencia clave: un socket es un tubo de bytes sin estructura; RPC agrega una capa encima que empaqueta "llamadas a función" con sus argumentos — Samba y NFS son, en el fondo, protocolos RPC especializados construidos sobre sockets.

## Conexión con el resto del curso

Ya viste Docker (Unidad 1) creando procesos aislados con su propia red; ahora Samba demuestra el caso contrario: dos sistemas operativos **distintos** (no solo procesos aislados del mismo kernel) compartiendo un recurso real a través de la red. La práctica de esta unidad ([02_practica_samba.md](02_practica_samba.md)) lo verifica de punta a punta con un servidor Samba real.

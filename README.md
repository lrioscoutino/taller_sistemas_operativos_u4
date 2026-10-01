# Unidad 4 — Interoperabilidad entre sistemas operativos

**Materia:** Taller de Sistemas Operativos (SCA-1026)
**Grupo:** S3B — Aula ED8
**Horario:** Lunes 17:00–19:00 · Jueves 19:00–21:00

Cierre del curso: después de administrar Linux (Unidad 1) y Windows Server (Unidad 2) por separado, esta unidad conecta ambos mundos — cómo un servidor Linux comparte recursos (archivos, impresoras) con clientes Windows, y viceversa, usando **Samba** como puente entre los dos sistemas de archivos en red.

## Contenido

- [01_interoperabilidad.md](01_interoperabilidad.md) — 4.1 Interoperabilidad: sistemas de archivos y recursos (NFS, SMB/Samba) · comunicación entre procesos (sockets, RPC)
- [02_practica_samba.md](02_practica_samba.md) — Práctica guiada: levantar un servidor Samba real, compartir un recurso y verificar interoperabilidad de punta a punta (servidor + cliente, protocolo SMB real)
- [03_actividades_evaluacion.md](03_actividades_evaluacion.md) — Actividades de aprendizaje y evaluación
- [04_proyecto_distros_seguridad.md](04_proyecto_distros_seguridad.md) — Proyecto de cierre por equipos: prueba de concepto de distribuciones Linux orientadas a seguridad y forense (Kali, CAINE, Tails, Qubes OS, entre otras)

## Nota sobre el código

La práctica de Samba se verificó con un servidor **real** (Samba 4.12 en Docker, no una simulación) y un cliente Python que habla el protocolo SMB de verdad — se listó, leyó y escribió un archivo cruzando la red, y se confirmó que el archivo llegó físicamente al disco del servidor. Los comandos de instalación nativa (`apt install samba`, `smbpasswd`) siguen la documentación oficial de Ubuntu/Debian.

## Fuentes

- Programa sinóptico oficial SCA-1026 — TecNM
- wiki.samba.org
- docs.docker.com (imagen `dperson/samba`)

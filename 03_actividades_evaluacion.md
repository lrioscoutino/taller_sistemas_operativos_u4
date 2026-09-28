# Actividades de aprendizaje y evaluación — Unidad 4

## Actividades de aprendizaje

- Instalar Samba de forma nativa (no en Docker) en una VM Linux propia, siguiendo el Paso 1, y compartir una carpeta real accesible desde otra máquina de la red del laboratorio.
- Conectarse al recurso compartido desde un cliente Windows real (Paso 6) y documentar con capturas el proceso completo: escribir la ruta UNC, ingresar credenciales, crear/leer un archivo.
- Repetir la práctica agregando un segundo usuario de solo lectura y verificar los permisos diferenciados.
- Investigar y documentar, en un cuadro comparativo, las diferencias entre NFS y SMB: en qué SO son nativos, qué puertos usan, cómo se administran los permisos.
- Investigar cómo se comparte una impresora vía Samba (`printable = yes` en `smb.conf`) y probarlo con una impresora virtual (PDF) si no hay una física disponible.
- Explorar `samba-tool` para configurar Samba como controlador de dominio de Active Directory, conectando el concepto con la administración de usuarios/grupos de Windows Server vista en la Unidad 2.

## Evaluación sugerida

| Evidencia | Qué valora |
|---|---|
| Servidor Samba instalado y compartiendo un recurso real | Ejecución real de la instalación y configuración (4.1.1), no solo lectura de los pasos |
| Conexión verificada desde un cliente distinto (Windows o Linux) | Comprobación real de interoperabilidad entre dos sistemas operativos |
| Cuadro comparativo NFS vs. SMB | Comprensión conceptual de 4.1.1 más allá del comando específico |
| Permisos diferenciados por usuario | Aplicación de configuración de seguridad de recursos compartidos |
| Reporte de `testparm`/`smbstatus` | Uso de herramientas reales de administración y diagnóstico |
| Examen de conceptos | Interoperabilidad, SMB/CIFS vs. NFS, sockets vs. RPC, rol de Samba en redes mixtas |

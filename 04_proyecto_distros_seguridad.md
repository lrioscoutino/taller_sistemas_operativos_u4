# Proyecto por equipos: Prueba de concepto — distribuciones Linux orientadas a seguridad y forense

Proyecto de cierre de curso. Cada equipo adopta **una distribución** orientada a ciberseguridad, pentesting o informática forense, instala una prueba de concepto real (no capturas de pantalla ajenas), y presenta tanto la teoría de la distribución como una demostración en vivo de al menos dos de sus herramientas insignia resolviendo un escenario concreto.

> El objetivo no es "usar Kali para escanear un puerto" — es entender **por qué existe cada distribución**, qué decisiones de diseño la hacen distinta de un Ubuntu con herramientas instaladas encima, y demostrarlo con las manos.

## Distribuciones sugeridas, por categoría

### Categoría 1 — Pentesting / Red Team (ataque ofensivo)

| Distribución | Base | Qué la distingue |
|---|---|---|
| **Kali Linux** | Debian | El estándar de la industria — +600 herramientas preinstaladas (Nmap, Metasploit, Burp Suite, Wireshark), mantenida por Offensive Security, usada en certificaciones (OSCP). |
| **Parrot Security OS** | Debian | Más ligera que Kali, con enfoque extra en anonimato (Tor/AnonSurf integrado) y "modo nube" para servidores. Buena para equipos con hardware limitado. |
| **BlackArch** | Arch Linux | +2800 herramientas — el catálogo más grande que existe. Instala como una capa de repositorios sobre un Arch existente, no como ISO independiente obligatoria. Exige más manejo de terminal (rolling release). |

### Categoría 2 — Informática forense (análisis post-incidente)

| Distribución | Base | Qué la distingue |
|---|---|---|
| **CAINE** (Computer Aided INvestigative Environment) | Ubuntu | Diseñada explícitamente para preservar la **cadena de custodia** de la evidencia — monta discos en modo solo-lectura por defecto, interfaz gráfica pensada para peritos no necesariamente técnicos. |
| **SIFT Workstation** (SANS) | Ubuntu | No es una ISO de arranque — es un conjunto de herramientas forenses (Autopsy, Volatility, Plaso) instalable sobre Ubuntu. Es lo que usan analistas profesionales en cursos del SANS Institute. |
| **Tsurugi Linux** | Ubuntu | Sucesora espiritual de DEFT (descontinuado) — fuerte en análisis de memoria, malware y dispositivos móviles. |

### Categoría 3 — Privacidad y anonimato (defensivo)

| Distribución | Base | Qué la distingue |
|---|---|---|
| **Tails** (The Amnesic Incognito Live System) | Debian | Corre **solo** desde USB en vivo, no deja rastro en el equipo host (amnésica por diseño), fuerza todo el tráfico de red a través de Tor. |
| **Qubes OS** | Fedora (dom0) + Xen | Seguridad por **compartimentación**: cada aplicación corre en su propia máquina virtual ligera, así que comprometer una (el navegador, ej.) no compromete las demás. Usado por Edward Snowden. |
| **Whonix** | Debian (sobre VirtualBox/KVM) | Separa en dos VMs — una "Gateway" que solo enruta tráfico por Tor, y una "Workstation" que no tiene acceso directo a la red — para que un error de configuración no revele la IP real. |

**Sugerencia de asignación:** con 6-9 equipos, asigna una distribución distinta a cada equipo, procurando que las tres categorías queden representadas — eso permite que la ronda de preguntas entre equipos compare pentesting vs. forense vs. privacidad al final.

## Estructura de la presentación (sugerida: 20-25 minutos por equipo)

### 1. Identidad de la distribución (3 min)
- ¿Quién la mantiene? ¿Desde cuándo existe? ¿Sigue activa (última versión, fecha)?
- ¿Sobre qué distribución base está construida, y por qué eligieron esa base?
- Licencia y costo (todas las sugeridas son gratuitas y de código abierto — verificarlo es parte del ejercicio).

### 2. El problema que resuelve (3 min)
- ¿Qué tarea sería *dolorosa* de hacer con un Ubuntu genérico, y esta distribución la resuelve de fábrica?
- Comparar explícitamente contra la categoría de al lado (ej. "a diferencia de Kali, CAINE prioriza no alterar la evidencia, no atacar").

### 3. Arquitectura y diseño (4 min)
- ¿Qué decisión técnica de diseño es la más distintiva? (ej. Qubes y sus VMs por aplicación; Tails y su modo amnésico; BlackArch y sus repos sobre Arch).
- Requisitos de hardware reales — ¿corre en una laptop de 2015? ¿Necesita virtualización anidada?

### 4. Catálogo de herramientas (3 min)
- Elegir **5 herramientas** representativas (no las 600 de Kali) y explicar en una frase qué hace cada una.
- Señalar al menos una herramienta que sea **exclusiva o especialmente mejor** en esa distribución frente a instalarla manualmente en otro Linux.

### 5. Prueba de concepto — demo en vivo (8-10 min, el corazón de la presentación)
- Instalar/arrancar la distribución real (VM, USB en vivo, o contenedor si la herramienta lo permite) — **frente al grupo**, no en video pregrabado.
- Ejecutar **dos herramientas distintas** resolviendo un escenario armado por el propio equipo, por ejemplo:
  - *Pentesting:* escanear una máquina víctima de laboratorio (ej. Metasploitable2, DVWA) con Nmap, y explotar una vulnerabilidad conocida con Metasploit.
  - *Forense:* analizar una imagen de disco de prueba (ej. del repositorio *Digital Corpora*) con Autopsy, encontrando un archivo borrado o un artefacto oculto.
  - *Privacidad:* arrancar Tails desde USB, verificar que el tráfico sale por Tor (ej. con `check.torproject.org`), y mostrar qué pasa al apagar la VM (nada persiste).
- **Nunca** contra sistemas que no sean del propio laboratorio o máquinas virtuales de práctica creadas para este fin — ver la nota de ética abajo.

### 6. Caso real / noticia relacionada (2 min)
- Un caso documentado en medios o en un reporte de seguridad donde esa distribución (o una de su misma categoría) haya sido mencionada — ej. el uso de Tails/Whonix por periodistas o activistas, Qubes recomendado por Snowden, un CTF o concurso donde ganó un equipo usando BlackArch.

### 7. Conclusiones del equipo (2 min)
- ¿Para qué tipo de organización o rol profesional recomendarían esta distribución?
- Una limitación o desventaja real que encontraron al probarla (no inventada — algo que de verdad les costó trabajo o no funcionó como esperaban).

## Entregables

1. Presentación (slides) siguiendo la estructura de arriba.
2. Demo en vivo — si algo falla en el momento, un video de respaldo de no más de 3 minutos grabado previamente por el propio equipo (no un video de YouTube de terceros).
3. Reporte corto (1-2 páginas) con: capturas del proceso de instalación, el escenario de la prueba de concepto, y las conclusiones del punto 7.

## Nota de ética y alcance — léela antes de empezar

Estas distribuciones incluyen herramientas ofensivas reales (escáneres, explotadores, crackers de contraseñas). El uso de estas herramientas **solo** está autorizado contra:

- Máquinas virtuales de laboratorio creadas específicamente para practicar (Metasploitable, DVWA, VulnHub, TryHackMe/HackTheBox en sus salas gratuitas).
- Imágenes de disco de prueba públicas y diseñadas para análisis forense educativo (ej. *Digital Corpora*, *NIST CFReDS*).
- Equipos propios del estudiante.

**Nunca** contra redes, dispositivos o cuentas de terceros, de la escuela, o de otros estudiantes, sin autorización explícita y por escrito. Usar estas herramientas contra sistemas sin autorización es un delito en México (Código Penal Federal, artículos sobre acceso ilícito a sistemas informáticos) y en la mayoría de los países — la línea entre "proyecto escolar" y "delito informático" es exactamente esa autorización.

### Ejemplos concretos — para que no quede ambigüedad

| ✅ SÍ está permitido | ❌ NO está permitido |
|---|---|
| Escanear con Nmap una Metasploitable2 corriendo en tu propia VM | Escanear la red del salón, de la escuela, o de tu casa compartida sin permiso explícito de cada dueño del equipo |
| Explotar una vulnerabilidad conocida de DVWA/VulnHub que tú mismo levantaste | Explotar el WiFi, router o cualquier dispositivo de un compañero, vecino o cafetería "para probar si se puede" |
| Analizar una imagen de disco descargada de *Digital Corpora* o *NIST CFReDS* | Analizar el celular, laptop o cuenta de redes sociales de otra persona sin su consentimiento por escrito |
| Resolver una sala gratuita de TryHackMe/HackTheBox con tu propia cuenta | Intentar acceder a sistemas de la escuela (portal de calificaciones, Wi-Fi institucional, servidores del TecNM) |
| Crackear un hash o contraseña que **tú mismo generaste** para la demo | Crackear una contraseña real de un compañero, de una cuenta institucional, o capturada de tráfico ajeno |
| Grabar la demo usando tus propias VMs como "víctima" y "atacante" | Usar la demo en vivo para atacar algo fuera del laboratorio controlado "ya que estamos" |

**Regla simple para decidir en el momento:** si el objetivo no es una VM/imagen/cuenta que tú mismo creaste o descargaste de un repositorio educativo explícitamente diseñado para esto, **no lo ataques** — pregúntale al profesor antes, no después.

## Evaluación sugerida

| Evidencia | Qué valora |
|---|---|
| Identidad y problema que resuelve (puntos 1-2) | Investigación real, no copiar la descripción de la página oficial sin entenderla |
| Arquitectura y diseño (punto 3) | Comprensión técnica de la decisión de diseño distintiva, no solo listar características |
| Demo en vivo funcionando (punto 5) | Evidencia de que de verdad instalaron y probaron la distribución — la parte que no se puede fingir |
| Manejo de un fallo en vivo | Qué tan bien el equipo explica y resuelve un problema imprevisto durante la demo (muy valorado: nadie espera perfección, sí capacidad de diagnóstico) |
| Caso real documentado (punto 6) | Capacidad de conectar la herramienta con su uso en el mundo real, con fuente verificable |
| Reporte entregado | Documentación clara del proceso, no solo repetición de lo presentado oralmente |
| Cumplimiento de la nota de ética | Evidencia de que el escenario de la demo usó solo objetivos autorizados (labs propios, no terceros) |

# Registro Priorizado de Vulnerabilidades (Fases 3 y 4)

- **Cliente:** Gadir Logística, S.L.
- **Auditora:** Natalia Alejo Pérez
- **Alcance:** `172.17.0.0/24` (In Scope) · 8 equipos activos (`172.17.0.3` a `172.17.0.10`)
- **Reglas de compromiso:** ningún hallazgo de este registro se ha explotado ni "probado". Todo se basa en detección de versión, enumeración sin credenciales y consulta de fuentes públicas.

> **Cómo leer este registro.** Encontrar un CVE asociado a una versión **no demuestra** que el servicio sea vulnerable. En cada ficha se separa lo **observado** (con evidencia), lo que **se sospecha** y lo que **queda pendiente de comprobar** (fase siguiente, solo con permiso del cliente).

---

## 0. Fuentes y fecha de consulta

| Dato | Fuente | Fecha de consulta |
|---|---|---|
| CVSS (con versión), CWE, rangos de versiones afectadas | NVD, API 2.0 (`services.nvd.nist.gov`) | 2026-11-22 |
| EPSS (probabilidad de explotación a 30 días) y percentil | FIRST, API pública (`api.first.org/data/v1/epss`) | 2026-11-22 (puntuación del día) |
| KEV: alta en el catálogo y uso en ransomware | CISA, `known_exploited_vulnerabilities.json`, catálogo versión **2026.11.20** | 2026-11-22 |
| Versión de paquete corregida por la distribución | Debian Security Tracker (`security-tracker.debian.org`) | 2026-11-22 |
| Técnicas y debilidades | MITRE ATT&CK y CWE | 2026-11-22 |

- Si un dato no está disponible se indica **n/d**.
- Los CVSS se indican **con su versión**. Cuando NVD publica varias valoraciones se citan todas.
- Las consultas están anotadas en el cuaderno (Entrada 20). Las cifras sustituyen a las "aproximadas" de borradores anteriores.

---

## 1. Criterio de priorización (regla explícita)

El cliente solo puede corregir unas pocas cosas este trimestre. El orden **no** sale de ordenar por CVSS, sino de esta regla, aplicada siempre igual:

1. **Bloque A, en el catálogo KEV** (explotación conocida). Dentro del bloque se ordena por:
   1. uso en campañas de **ransomware = Known**;
   2. si la versión observada **cae dentro del rango afectado** y con qué evidencia;
   3. impacto (¿ejecución de código sin autenticar?) y, a igualdad, EPSS.
2. **Bloque B, fuera de KEV.** Se ordena por la **evidencia directa** del riesgo (servicio realmente expuesto frente a coincidencia solo de banner), luego EPSS y luego CVSS.
3. **Hallazgos de configuración sin CVE** (carpeta abierta, etc.) se ordenan por el impacto evidenciado. Si corregirlos cuesta 0 €, se recomienda hacerlo **ya**, independientemente de su puesto.

**Consecuencia práctica:** un fallo con CVSS 8.1 en KEV (VULN-04) va por delante de otro con CVSS 9.8 sin explotación conocida (VULN-06). Un CVSS 7.5 con servicio realmente expuesto (VULN-05) va por delante de un 9.8 que solo coincide por banner.

---

## 2. Tabla resumen priorizada

| Prio. | ID | Activo | Servicio / versión observada | CVE | CVSS (versión) | EPSS (percentil) | KEV (alta · ransomware) | Estado |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| **P1** | VULN-01 | `172.17.0.5:61616` | ActiveMQ OpenWire **5.15.15** | **CVE-2023-46604** | **10.0** (v3.1)* | 99.992 % (P99.986) | **Sí** · 2023-11-02 · **Known** | Versión afectada · no confirmada |
| **P2** | VULN-02 | `172.17.0.6:8080` | Jenkins **2.441** | **CVE-2024-23897** | **9.8** (v3.1) | 99.999 % (P99.995) | **Sí** · 2024-08-19 · **Known** | Versión afectada · no confirmada |
| **P3** | VULN-03 | `172.17.0.4:8009` | Tomcat **9.0.30** / AJP13 | **CVE-2020-1938** (Ghostcat) | **9.8** (v3.1) | 99.270 % (P99.938) | **Sí** · 2022-03-03 · Unknown | Versión afectada · no confirmada |
| **P4** | VULN-04 | `172.17.0.7:22` | OpenSSH 7.7 (sin sufijo de distribución) | **CVE-2026-123158** ("sshole") | **8.1** (v3.1) | 95.538 % (P99.871) | **Sí** · 2026-11-09 · Unknown | **Condicional:** depende de la distribución de `.7` |
| **P5** | VULN-05 | `172.17.0.10:623/udp` | IPMI **2.0** (RMCP) | **CVE-2013-4786** | **7.5** (v3.0) | 98.385 % (P99.919) | No | Exposición **confirmada** · fallo no comprobado |
| **P6** | VULN-06 | `172.17.0.3:21` | vsftpd **2.3.4** | **CVE-2011-2523** | **9.8** (v3.1) | 96.184 % (P99.879) | No | Solo coincide el banner |
| **P7** | VULN-07 | `172.17.0.8:445,139` | Samba (versión no determinada) | Sin CVE (configuración) | **5.3** (v3.1)** | n/a | n/a | **Confirmado** (acción de coste 0) |
| **P8** | VULN-08 | `172.17.0.7:22` | OpenSSH 7.7 | **CVE-2018-15473** | **5.3** (v3.1) | 98.675 % (P99.925) | No | Versión afectada · no confirmada |

\* NVD publica dos valoraciones v3.1 de este CVE: 10.0 (`S:C`) y 9.8. Se usa la más alta como referencia y se citan ambas en la ficha.
\*\* Escenario evaluado con la evidencia disponible (token caducado). Ver VULN-07 para el escenario peor (8.2).

### Por qué este orden

- **P1 antes que P2.** Ambos están en KEV con ransomware **Known** y EPSS ≈ 99.99 %. ActiveMQ permite **ejecución de código sin autenticar** y la versión se ha leído del propio protocolo del broker. Jenkins devuelve **403** en la raíz (ver VULN-02): sin lectura anónima, el alcance del fallo podría ser menor.
- **P3.** En KEV pero sin ransomware conocido; la versión (9.0.30 < 9.0.31) está confirmada dentro del rango.
- **P4.** En KEV y reciente, pero solo aplica a Linux con **musl sin PAM**. Es el único caso donde la versión sola no basta: hay que conocer la distribución de `.7`.
- **P5 antes que P6.** IPMI tiene el servicio **realmente expuesto** (se vio el puerto y la versión del protocolo) y es una interfaz de administración de muy alto privilegio. vsftpd solo coincide por banner y el backdoor original no está en los paquetes de las distribuciones.
- **P7.** Confirmado y de **coste cero**: hay que corregirlo esta semana, aunque su puntuación sea media.

> **Aviso inmediato de hallazgos muy graves (regla del cliente: "anótalo y avísalo").** VULN-01, VULN-02 y VULN-03 se comunican a la Gerencia con carácter inmediato en el informe. Ninguno se ha tocado ni "comprobado" explotándolo.

---

## 3. Fichas detalladas

### VULN-01 · Ejecución remota de código en Apache ActiveMQ (CVE-2023-46604)

- **Activo:** `172.17.0.5:61616/tcp` (broker de mensajería; probable "mensajería de los operadores" del correo del cliente, hipótesis pendiente de confirmar con el cliente).
- **CWE:** CWE-502 (Deserialización de datos no confiables), según NVD.
- **Técnica MITRE ATT&CK:** T1190 (Exploit Public-Facing Application); tras la ejecución, T1059 (Command and Scripting Interpreter).
- **CVSS v3.1:** **10.0** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:L/I:H/A:H` (NVD también publica 9.8 `…/S:U/C:H/I:H/A:H`).
  - **Confidencialidad:** baja en la valoración de 10.0 y alta en la de 9.8; en la práctica, quien ejecuta comandos puede leer lo que el servicio y el sistema ven.
  - **Integridad:** alta (puede modificar mensajes y el sistema).
  - **Disponibilidad:** alta (puede detener el broker o el equipo).
  - El alcance cambia (`S:C`) porque el broker puede servir de salto a otros equipos.
- **EPSS:** 99.992 % (percentil 99.986), 2026-11-22.
- **CISA KEV:** **Sí**, alta 2023-11-02, uso en ransomware **Known**.
- **Descripción:** el marshaller del protocolo OpenWire permite a un atacante con acceso de red al broker ejecutar comandos manipulando clases serializadas.
- **Evidencias observadas** (enlaces a los ficheros):
  - `nmap -sV` sobre los puertos altos de `.5`: `61616/tcp apachemq ActiveMQ OpenWire transport 5.15.15` → [servicios_172.17.0.5_altos.txt](evidencias/salidas-laboratorio/servicios_172.17.0.5_altos.txt) (Entrada 16).
  - `61613/tcp` responde como STOMP de ActiveMQ; `8161/tcp` y `61614/tcp` son Jetty 9.4.39; también abiertos `1883` (MQTT), `5672` (AMQP) y `46335` (`tcpwrapped`, no identificado).
  - `curl -I http://172.17.0.5:8161/` → `401` con `realm="ActiveMQRealm"` (consola de ActiveMQ que exige autenticación).
  - Mapa TCP completo → [tcp_completo_172.17.0.3-9.txt](evidencias/salidas-laboratorio/tcp_completo_172.17.0.3-9.txt) (Entrada 15). Estos puertos **no aparecían** en el primer escaneo (solo los 1000 más comunes).
- **Por qué se sospecha:** NVD marca como afectada la rama 5.15 **anterior a 5.15.16**; la versión observada es 5.15.15. La versión la anuncia el propio protocolo del broker, no un banner de la distribución.
- **Limitaciones (reglas de compromiso):** no se envió ningún mensaje OpenWire manipulado ni se usó ninguna comprobación que interactúe con el fallo. No se probaron credenciales en la consola `8161`.
- **Pendiente de comprobar (fase siguiente, con permiso):** confirmar que el broker es explotable; confirmar con el cliente que es el servidor de mensajería.
- **Remediación:** actualizar a una versión corregida (mínimo 5.15.16, 5.16.7, 5.17.6 o 5.18.3 según la rama; mejor una rama soportada actual), limitar `61616` a los equipos que lo necesiten y revisar que la consola `8161` no use credenciales por defecto.
- **Relacionado, no priorizado:** ActiveMQ tiene también **CVE-2026-34197** (KEV 2026-04-16, ransomware Unknown, CVSS 8.8 v3.1, EPSS 98.256 % P99.917, versiones anteriores a 5.19.4 → `5.15.15` queda dentro del rango por versión). Exige iniciar sesión en la consola, así que **no se puede comprobar** sin probar credenciales. Queda pendiente.

---

### VULN-02 · Lectura arbitraria de ficheros en Jenkins (CVE-2024-23897)

- **Activo:** `172.17.0.6:8080` (Jenkins 2.441 sobre Jetty 10.0.18).
- **CWE:** CWE-22 / CWE-27 (Path traversal), según NVD.
- **Técnica MITRE ATT&CK:** T1190 (Exploit Public-Facing Application); T1552.001 (Credentials In Files) si se leen secretos.
- **CVSS v3.1:** **9.8** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`.
  - **C:** alta (lectura de ficheros como claves y secretos de Jenkins).
  - **I:** alta (con esos secretos se puede llegar a ejecutar código en el servidor de integración continua).
  - **D:** alta.
- **EPSS:** 99.999 % (percentil 99.995). **KEV:** **Sí**, 2024-08-19, ransomware **Known**.
- **Descripción:** la interfaz de línea de comandos de Jenkins expande argumentos que empiezan por `@` leyendo ficheros locales. NVD marca afectadas las versiones anteriores a **2.442** (y LTS anteriores a **2.426.3**).
- **Evidencias observadas:** cabecera `X-Jenkins: 2.441` y servidor Jetty 10.0.18 (Entrada 08, [registro_sesion_terminal.log](evidencias/registro_sesion_terminal.log)); puerto 8080 como único puerto web de `.6` (Entrada 15).
- **Matiz importante:** la raíz responde **403**, es decir, no hay lectura anónima general. Según el aviso del proveedor, sin ese permiso el atacante solo alcanzaría las primeras líneas de los ficheros, no el contenido completo. El 9.8 describe el peor caso.
- **Limitaciones:** no se envió ninguna petición al CLI de Jenkins ni se intentó leer ningún fichero.
- **Pendiente de comprobar:** si el acceso anónimo está activado (determina el impacto real).
- **Remediación:** actualizar a 2.442 o superior (rama semanal) o a LTS 2.426.3 o superior; si no es posible, deshabilitar el acceso al CLI y limitar el puerto 8080 por cortafuegos.

---

### VULN-03 · Ghostcat: lectura de ficheros vía AJP en Apache Tomcat (CVE-2020-1938)

- **Activo:** `172.17.0.4:8009/tcp` (AJP13) y `:8080` (Tomcat 9.0.30).
- **CWE:** NVD no asigna una CWE concreta (`NVD-CWE-Other`). Clasificación propia: **CWE-200** (exposición de información sensible).
- **Técnica MITRE ATT&CK:** T1190; T1005 (Data from Local System).
- **CVSS v3.1:** **9.8** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`.
  - **C:** alta (descarga de ficheros de la aplicación, p. ej. `web.xml` y configuración).
  - **I:** alta (posible ejecución de código si la aplicación admite subida de ficheros).
  - **D:** alta.
- **EPSS:** 99.270 % (percentil 99.938). **KEV:** **Sí**, 2022-03-03, ransomware Unknown.
- **Descripción:** el conector AJP viene habilitado y sin secreto en versiones anteriores a **9.0.31** de la rama 9. NVD lo confirma para el rango 9.0.0 a 9.0.30.
- **Evidencias observadas:** `8009/tcp open ajp13 Apache Jserv (Protocol v1.3)` y `8080/tcp Apache Tomcat 9.0.30` (Entradas 03 y 15); título `Apache Tomcat/9.0.30` (Entrada 08).
- **Limitaciones:** no se enviaron paquetes AJP manipulados ni se pidió ningún fichero. No se probó `/manager` ni credenciales.
- **Pendiente de comprobar:** si el conector AJP responde sin secreto y qué ficheros expone.
- **Remediación:** actualizar a la última 9.0.x, desactivar el conector AJP si no se usa o definir un secreto, y limitar el puerto `8009`.
- **Candidato secundario:** por versión, `9.0.30` también cae en el rango de **CVE-2025-24813** (KEV 2025-04-01, anterior a 9.0.99). Su explotación depende de una configuración no habitual, así que queda **pendiente** (sección 4).

---

### VULN-04 · Salto de autenticación en OpenSSH, "sshole" (CVE-2026-123158)

- **Activo:** `172.17.0.7:22/tcp` (OpenSSH 7.7). Los otros cuatro servidores SSH **no se esperan afectados** (ver abajo).
- **CWE:** CWE-394 (Código de estado inesperado), según CISA/NVD.
- **Técnica MITRE ATT&CK:** T1021.004 (Remote Services: SSH); T1078 (Valid Accounts).
- **CVSS v3.1:** **8.1** `CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:H` (la complejidad alta refleja que solo ocurre en ciertas configuraciones).
  - **C / I / A:** altas (acceso como una cuenta existente).
- **EPSS:** 95.538 % (percentil 99.871). **KEV:** **Sí**, 2026-11-09, ransomware Unknown, CISA indica triaje forense.
- **Descripción (NVD):** `sshd` de OpenSSH **anterior a 10.6p2**, en **Linux con musl libc y sin PAM** (ejemplo: Alpine), carece de una mitigación frente a un salto de autenticación por contraseña: en algunas configuraciones de cuenta, **cualquier contraseña de entre 256 y 1024 caracteres** es aceptada.
- **Evidencias observadas (los 5 servidores SSH, Entradas 03 y 15):**

  | Equipo | Banner | Lectura |
  |---|---|---|
  | `.4` | OpenSSH 7.9p1 **Debian** 10+deb10u4 | Debian (glibc, con PAM): no se espera afectado |
  | `.6` | OpenSSH 9.2p1 **Debian** 2+deb12u2 | Debian: no se espera afectado |
  | `.7` | OpenSSH **7.7** (sin sufijo de distribución) | **Distribución desconocida; no se puede descartar** |
  | `.8` | OpenSSH 9.6p1 **Ubuntu** 3ubuntu13.5 | Ubuntu: no se espera afectado |
  | `.9` | OpenSSH 9.2p1 **Debian** 2+deb12u10 | Debian: no se espera afectado |

- **Razonamiento:** todas las versiones son numéricamente "anteriores a 10.6p2", pero eso **no basta**: el fallo requiere musl y ausencia de PAM. Los banners con sufijo Debian/Ubuntu indican paquetes de esas distribuciones (glibc y PAM), **lo cual es una deducción, no una comprobación**. Alpine no añade sufijo al banner, igual que `.7`.
- **Limitaciones:** comprobarlo exigiría enviar contraseñas largas, es decir, **probar credenciales**, lo que las reglas prohíben. No se hizo.
- **Pendiente de comprobar:** identificar la distribución y la compilación de OpenSSH de `.7` (preguntando al cliente o con una vía no intrusiva).
- **Remediación:** actualizar a OpenSSH 10.6p2 o superior (o aplicar las mitigaciones del proveedor) y restringir el puerto 22 por cortafuegos a las IP de administración.

---

### VULN-05 · Interfaz de gestión IPMI expuesta (CVE-2013-4786)

- **Activo:** `172.17.0.10:623/udp` (RMCP, IPMI **2.0**).
- **CWE:** CWE-255 (Gestión de credenciales), según NVD.
- **Técnica MITRE ATT&CK:** T1110.002 (Brute Force: Password Cracking, descifrado offline del material obtenido).
- **CVSS v3.0:** **7.5** `CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N`. NVD solo publica v3.0 y v2 (7.8) para este CVE.
  - **C:** alta (permite obtener material con el que adivinar contraseñas fuera de línea).
  - **I / A:** no puntúan en el CVSS, pero el impacto real es mayor: un BMC controla encendido, consola serie (SOL) y firmware del servidor.
- **EPSS:** 98.385 % (percentil 99.919). **KEV:** No.
- **Descripción:** en IPMI 2.0, el intercambio de autenticación RAKP permite a un atacante remoto obtener el HMAC de la respuesta del BMC y probar contraseñas offline.
- **Evidencias observadas:**
  - `623/udp open asf-rmcp` en `.10` y **cerrado** en los otros 7 equipos → [escaneo_udp_623.txt](evidencias/salidas-laboratorio/escaneo_udp_623.txt) (Entrada 12) y [udp_top100.txt](evidencias/salidas-laboratorio/udp_top100.txt) (Entrada 18).
  - `ipmi-version` (script `discovery safe`): `Version: IPMI-2.0`, `Level: 1.5, 2.0`, `UserAuth: password, md5, md2, null`, `PassAuth: auth_msg, auth_user, non_null_user` → [ipmi_version_172.17.0.10.txt](evidencias/salidas-laboratorio/ipmi_version_172.17.0.10.txt) (Entrada 13). Se registra la salida **literal**; no se interpreta el valor `null` sin confirmarlo en la documentación del script.
  - La página web de `.10` es documentación técnica sobre IPMI/BMC y menciona este CVE → [ipmi_172.17.0.10.html](evidencias/salidas-laboratorio/ipmi_172.17.0.10.html) (Entrada 10). Es contexto, **no** evidencia del servicio.
  - Este puerto **no estaba** en el primer escaneo UDP (`--top-ports 20`).
- **Limitaciones:** confirmar el fallo exige enviar el intercambio RAKP, o usar scripts como `ipmi-cipher-zero` o módulos de Metasploit, que interactúan con la vulnerabilidad. **No se hizo.**
- **Pendiente de comprobar (con permiso):** si este BMC concreto devuelve el material en el intercambio RAKP.
- **Remediación:** aislar el BMC en una red de gestión restringida (cortafuegos o VLAN), credenciales robustas e individuales, actualizar el firmware y desactivar IPMI sobre LAN si no se usa.

---

### VULN-06 · Posible puerta trasera en vsftpd 2.3.4 (CVE-2011-2523)

- **Activo:** `172.17.0.3:21/tcp`.
- **CWE:** CWE-78 (Inyección de comandos del SO), según NVD. Es habitual citar también CWE-506 (código malicioso embebido) por su origen.
- **Técnica MITRE ATT&CK:** T1190; T1059.004 (Unix Shell).
- **CVSS v3.1:** **9.8** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`. **C/I/A:** altas (shell de superusuario en caso de existir la puerta trasera).
- **EPSS:** 96.184 % (percentil 99.879). **KEV:** No.
- **Descripción:** el paquete fuente de vsftpd 2.3.4 distribuido en el sitio oficial a comienzos de julio de 2011 incluía una puerta trasera.
- **Evidencias observadas:** `21/tcp open ftp vsftpd 2.3.4` (Entradas 03 y 15).
- **Importante:** **el banner no prueba la puerta trasera.** Solo el paquete comprometido de 2011 la llevaba; los paquetes de las distribuciones no. NVD registra "vsftpd 2.3.4" como versión afectada, pero la coincidencia es solo de versión. Por eso queda como "posible".
- **Limitaciones:** no se envió el disparador (usuario con `:)`) y no se usó el script de Nmap `ftp-vsftpd-backdoor`, que pertenece a categorías prohibidas. No se probó el acceso anónimo.
- **Pendiente de comprobar (con permiso):** si la puerta trasera está presente.
- **Remediación:** sustituir por una versión soportada de vsftpd, o por SFTP, y limitar el acceso al puerto 21.

---

### VULN-07 · Carpeta compartida abierta y enumeración anónima en Samba

- **Activo:** `172.17.0.8` (puertos `139/445` y `137/udp`).
- **CWE:** CWE-284 (Control de acceso incorrecto); CWE-200 (Exposición de información, por la enumeración anónima de cuentas); CWE-312 (secreto en un fichero accesible).
- **Técnica MITRE ATT&CK:** T1135 (Network Share Discovery); T1087.001 (Account Discovery: Local Account); T1552.001 (Credentials In Files).
- **CVSS v3.1 (escenario evaluado):** **5.3** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N`.
  - **C:** baja, porque lo expuesto es la identidad de cuentas y un token **caducado**.
  - **I / A:** ninguna.
  - **Escenario peor:** si el token siguiera siendo válido, `C:H/I:L` daría **8.2**. No hay evidencia de ello.
- **EPSS / KEV:** n/a (no es un CVE).
- **Evidencias observadas** ([enum_samba_172.17.0.8.txt](evidencias/salidas-laboratorio/enum_samba_172.17.0.8.txt), [registro_sesion_terminal.log](evidencias/registro_sesion_terminal.log), Entradas 05 a 07 y 17):
  - Sesión nula permitida (`allows sessions using username '', password ''`); grupo de trabajo `ESEEMEB.DL`, nombre NetBIOS `SAMBASERVER`.
  - Recurso `myshare`: listado anónimo correcto; `backup24` y `home`: acceso denegado.
  - Cuentas obtenidas sin autenticar: `usuario1`, `usuario2`, `usuario3`, `satriani7`, `administrador` (RID 1000 a 1004), `nobody` (RID 501) y el usuario Unix `ubuntu` (UID 1000).
  - `access.txt` (956 bytes) con un JWT firmado RS256. Payload decodificado: `email satriani7@eseemeb.dl`, `role user`, `iat 2024-10-05 20:32:53 UTC`, **`exp 2024-10-05 21:32:53 UTC`** (caducado hace más de dos años) y una clave pública RSA incrustada (`jwk`).
- **Limitaciones:** no se usó el token ni se probó contra el portal de `.8:80`. **La versión exacta de Samba no se pudo determinar** (`smb-os-discovery` no devolvió información; `srvinfo` solo indica "Samba Server"). Por eso **SambaCry (CVE-2017-7494, KEV, rango 3.5.0 a 4.6.3) no se puede descartar por versión**; queda en la sección 4. Una inferencia débil: `.8` parece Ubuntu 24.04 por su banner SSH, que trae una rama de Samba muy posterior.
- **Pendiente de comprobar:** con el portal, si confía en la clave `jwk` incrustada en el token (decisión del equipo del cliente, no de esta auditoría).
- **Remediación (coste 0):** borrar `access.txt`, desactivar el acceso anónimo y las sesiones nulas, exigir autenticación en todos los recursos.

---

### VULN-08 · Enumeración de usuarios en OpenSSH 7.7 (CVE-2018-15473)

- **Activo:** `172.17.0.7:22/tcp`.
- **CWE:** CWE-362 según NVD; la debilidad observable es la discrepancia de respuesta (equivalente a CWE-203).
- **Técnica MITRE ATT&CK:** T1087.001 (Account Discovery: Local Account).
- **CVSS v3.1:** **5.3** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` (NVD, valoración principal; la secundaria es 5.9).
  - **C:** baja (revela qué usuarios existen). **I / A:** ninguna.
- **EPSS:** 98.675 % (percentil 99.925). **KEV:** No.
- **Descripción:** OpenSSH hasta 7.7 permite saber si un usuario existe por diferencias en la respuesta a una autenticación truncada.
- **Evidencias observadas:** `22/tcp open ssh OpenSSH 7.7 (protocol 2.0)` (Entradas 03 y 15).
- **Importante:** aunque la puntuación es media, su EPSS es muy alto. No está en KEV, por lo que queda por detrás de los hallazgos con explotación conocida.
- **Limitaciones:** comprobarlo exige enviar paquetes de autenticación manipulados. No se hizo.
- **Remediación:** actualizar OpenSSH a 7.8 o superior (idealmente 10.6p2 o superior, por VULN-04).

---

## 4. Candidatos secundarios (no priorizados: pendientes de contrastar)

| Activo | CVE | KEV · ransomware | Por qué es candidato | Qué falta para decidir |
|---|---|---|---|---|
| `.4` Tomcat 9.0.30 | CVE-2025-24813 (CVSS 9.8, EPSS 99.942 %) | Sí · Unknown | Versión anterior a 9.0.99 | Depende de una configuración no habitual (escritura en el servlet por defecto) |
| `.5` Apache 2.4.59 (Debian) | CVE-2024-38475 (CVSS 9.1, EPSS 99.957 %) | Sí · Unknown | Debian corrigió rebasando a ≥ 2.4.61; un banner 2.4.59 indica que **no** se aplicó | Que existan reglas `mod_rewrite` afectadas |
| `.3`, `.8` Apache 2.4.58 (Ubuntu) | CVE-2024-38475 | Sí · Unknown | Versión anterior a 2.4.60 | Ubuntu puede haberlo corregido por *backport*; no verificado |
| `.6` OpenSSH 9.2p1 Debian **2+deb12u2** | CVE-2024-6387, "regreSSHion" (CVSS 8.1, EPSS 99.506 %) | No | Debian lo corrigió en **2+deb12u3**: `.6` es **anterior al parche** | Complejidad alta (condición de carrera). Se confirma con el *tracker* de Debian, no explotando |
| `.8` OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 | CVE-2024-6387, "regreSSHion" | No | La versión (9.6p1) cae dentro del rango de OpenSSH afectado (de 8.5p1 a anterior a 9.8p1) | Si Ubuntu corrigió el paquete por *backport*: **no consultado** el *tracker* de Ubuntu |
| `.8` Samba | CVE-2017-7494 "SambaCry" (CVSS 9.8, EPSS 99.448 %) | Sí · **Known** | Versión no determinada | Versión exacta y recurso con escritura (el listado mostró "Writing: N/A") |

---

## 5. Descarte razonado de falsos positivos

Se descartan por **versión fuera de rango** o **parche de la distribución**, no por intuición:

| Servicio | CVE | KEV | Motivo del descarte | Fuente |
|---|---|---|---|---|
| ActiveMQ 5.15.15 (`.5`) | CVE-2016-3088 | Sí | Afecta a 5.x **anterior a 5.14.0** | NVD |
| Jenkins 2.441 (`.6`) | CVE-2017-1000353 | Sí | Afecta a ≤ 2.56 (LTS ≤ 2.46.1) | NVD |
| Jenkins 2.441 (`.6`) | CVE-2018-1000861 | Sí | Afecta a ≤ 2.153 (LTS ≤ 2.138.3) | NVD |
| Apache 2.4.58/2.4.59/2.4.68 | CVE-2021-41773 | Sí | Solo afecta a **2.4.49** | NVD |
| OpenSSH 7.9p1 (`.4`) y 7.7 (`.7`) | CVE-2024-6387 | No | El código vulnerable se introdujo en la **8.5p1** | Debian Tracker |
| OpenSSH 9.2p1 `2+deb12u10` (`.9`) | CVE-2024-6387 | No | Versión de paquete **posterior** al parche `2+deb12u3` | Debian Tracker |
| Apache 2.4.68 Debian (`.9`) | CVE-2024-38475 | Sí | Versión posterior a la corrección (≥ 2.4.61) | Debian Tracker |
| Tomcat 9.0.30 (`.4`) | CVE-2026-34486 | Sí | Afecta solo a 9.0.116 (también 10.1.53 y 11.0.20); EPSS 98.503 % | NVD, FIRST |
| IPMI en `.3` a `.9` | CVE-2013-4786 | No | El puerto `623/udp` está **cerrado** en esos equipos | Entradas 12 y 18 |

**Datos erróneos del inventario del cliente (no son falsos positivos de una herramienta, sino activos que no existen como se describían):** IIS 7.5 y SQL Server 2008 (`.10`/`.11`), Windows 7 con RDP y BlueKeep (`.9`), MSMQ en Windows Server 2003 y Windows XP con FileZilla. `.9` y `.10` son Linux, y el mapa TCP completo no muestra abiertos los puertos 1433, 3389 ni 1801 en ningún equipo. Detalle en [inventario.md](inventario.md).

**No se ha usado ningún escáner automático de vulnerabilidades:** la correlación versión → CVE es manual, contra NVD, EPSS, KEV y el *tracker* de Debian.

---

## 6. Cobertura por servicio (de la versión al CVE)

Todos los servicios con versión del [inventario](inventario.md) y su resultado, para dejar claro que ninguno se ha quedado sin revisar. "Sin entradas en KEV" significa que no hay ningún CVE de ese producto en el catálogo 2026.11.20 (búsqueda por producto en el JSON).

| Equipo · puerto | Servicio y versión | Resultado |
|---|---|---|
| `.3` · 21 | vsftpd 2.3.4 | **VULN-06** |
| `.3` · 80 y `.8` · 80 | Apache httpd 2.4.58 (Ubuntu) | Candidato CVE-2024-38475 (sección 4). El resto de CVE de Apache en KEV no aplica (p. ej. CVE-2021-41773 solo afecta a 2.4.49) |
| `.4` · 22 | OpenSSH 7.9p1 Debian 10+deb10u4 | **Sin hallazgo:** CVE-2018-15473 afecta a ≤ 7.7 y CVE-2024-6387 a ≥ 8.5p1 |
| `.4` · 8009 y 8080 | Tomcat 9.0.30 con AJP13 | **VULN-03**; candidato CVE-2025-24813; descartado CVE-2026-34486 |
| `.5` · 80 y 443 | Apache httpd 2.4.59 (Debian) | Candidato CVE-2024-38475 (Debian lo corrigió en ≥ 2.4.61) |
| `.5` · 61616, 61613, 8161 | ActiveMQ OpenWire 5.15.15, STOMP, consola web | **VULN-01**; candidato CVE-2026-34197; descartado CVE-2016-3088 |
| `.5` · 8161 y 61614 | Jetty 9.4.39.v20210325 | Sin entradas de Jetty en KEV. Se trata como componente de ActiveMQ |
| `.5` · 1883 y 5672 | MQTT y AMQP, sin versión identificada | **No se puede asociar un CVE sin versión.** Pendiente de identificar |
| `.5` · 46335 | `tcpwrapped` | **No identificado** |
| `.6` · 22 | OpenSSH 9.2p1 Debian 2+deb12u2 | Candidato CVE-2024-6387 (anterior al parche de Debian) |
| `.6` · 8080 | Jenkins 2.441 sobre Jetty 10.0.18 | **VULN-02**; descartados CVE-2017-1000353 y CVE-2018-1000861; Jetty sin entradas en KEV |
| `.7` · 22 | OpenSSH 7.7 | **VULN-04** (condicional) y **VULN-08**; CVE-2024-6387 fuera de rango |
| `.8` · 22 | OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 | Candidato CVE-2024-6387 (corrección de Ubuntu no consultada) |
| `.8` · 139, 445 y 137/udp | Samba smbd 4 (versión exacta no determinada), NetBIOS | **VULN-07**; candidato CVE-2017-7494 |
| `.8` · 80 | Portal de login (aplicación propia) | **Sin versión identificable**: no se puede asociar un CVE. No se enviaron datos al formulario |
| `.9` · 22 | OpenSSH 9.2p1 Debian 2+deb12u10 | **Sin hallazgo:** versión de paquete posterior al parche de Debian |
| `.9` · 80 | Apache httpd 2.4.68 (Debian) | **Sin hallazgo:** posterior a la corrección de CVE-2024-38475 |
| `.10` · 80 | Python SimpleHTTP 3.11.2 (página documental) | Sin entradas en KEV |
| `.10` · 623/udp | IPMI 2.0 | **VULN-05** |

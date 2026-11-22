# Cuaderno de trabajo · Natalia Alejo Pérez
**Proyecto:** El informático fantasma · Gadir Logística, S.L.  
**Alcance legal:** `172.17.0.0/24` (definido en `scope.txt`)  
**Reglas de compromiso:** Solo identificación y enumeración sin credenciales. Cero explotación, cero ataques intrusivos o de fuerza bruta. Sin interacción destructiva sobre equipos delicados.

---

## Metodología de registro (cinco notas de la UD1)
Cada comprobación se registra con las cinco notas:
1. **Cuándo:** marca temporal exacta (hora del `date` o de la cabecera de Nmap, en UTC).
2. **Objetivo y en alcance:** activo evaluado y verificación de pertenencia a `172.17.0.0/24`.
3. **Qué se hizo:** comando exacto, con flags y justificación no intrusiva.
4. **Qué salió:** evidencia y salidas de terminal.
5. **Siguiente paso:** deducción lógica y siguiente comprobación.

**Registros brutos de sesión** (`script`), que permiten comprobar cada hora y cada comando:
- Sesión 1 (17:03:13 a 17:22:42 UTC): [`evidencias/registro_sesion_terminal.log`](../evidencias/registro_sesion_terminal.log)
- Sesiones 2 y 3 (17:58:20 a 17:59:43 y 18:01:51 a 18:55:22 UTC): [`evidencias/salidas-laboratorio/registro_sesion_terminal_2.log`](../evidencias/salidas-laboratorio/registro_sesion_terminal_2.log)
- Salidas de cada escaneo: carpeta [`evidencias/salidas-laboratorio/`](../evidencias/salidas-laboratorio/)

**Convenciones**
- Todas las horas están en **UTC**. La captura `01_conexion_ssh_exitosa.png` muestra la hora local peninsular (CET, UTC+1).
- Las comprobaciones de lectura sobre ficheros locales (Entradas 17) no generan tráfico de red.
- "Técnica activa no intrusiva" significa que se envían paquetes a los equipos (por eso **no** es pasiva), pero sin explotar, sin credenciales y sin scripts de categorías prohibidas.

---

## Entrada 00 · Conexión al laboratorio
- **Cuándo:** 2026-11-22 ≈16:55:47 UTC (17:55:47 hora local, según la captura [`01_conexion_ssh_exitosa.png`](../evidencias/01_conexion_ssh_exitosa.png)).
- **Objetivo y en alcance:** acceso a la máquina de auditoría `alumno20` (Kali del laboratorio) a través de `172.26.0.55:2220`. **No es un objetivo de la auditoría**, solo el punto de entrada al laboratorio; los objetivos son `172.17.0.0/24`.
- **Qué se hizo:**
  ```powershell
  ssh -p 2220 alumno@172.26.0.55
  ```
  `-p 2220` es el puerto SSH del laboratorio; se pide la contraseña del laboratorio.
- **Qué salió:** sesión abierta en `alumno@alumno20` (`Bienvenido a tu Kali de laboratorio`). El aviso `/etc/machine-id: Permission denied` es del propio laboratorio y no afecta.
- **Siguiente paso:** iniciar el registro de sesión con `script`, anotar la hora y comprobar la red de la máquina de auditoría.

---

## Entrada 01 · Verificación de interfaz y fijación del alcance
- **Cuándo:** 2026-11-22 17:03:25 UTC (`date` del propio log).
- **Objetivo y en alcance:** verificar la red de la máquina de auditoría (`alumno20`) y confirmar su dirección en el segmento autorizado `172.17.0.0/24`.
- **Qué se hizo:**
  ```bash
  date
  ip a
  ```
  `date` fija la hora; `ip a` lista las interfaces.
- **Qué salió:**
  - `docker0` con `172.17.0.1/16`: es la **propia máquina de auditoría** y le da conectividad directa con el segmento de Gadir. El alcance autorizado es solo el `/24`, aunque la interfaz sea un `/16`.
  - Otras interfaces: `eth0` `10.30.0.30/16` y `eth1` `10.50.0.120/24` (redes del laboratorio, **fuera del alcance**; no se escanearon) y varias `veth` de los contenedores.
- **Evidencia:** [`02_ip_a_kali_lab.jpg`](../evidencias/02_ip_a_kali_lab.jpg)
  ```text
  3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
      inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
  ```
- **Siguiente paso:** descubrimiento de equipos vivos sobre `172.17.0.0/24`, sin tocar puertos.

---

## Entrada 02 · Descubrimiento de equipos vivos
- **Cuándo:** 2026-11-22 17:04 UTC (cabecera de Nmap: `at 2026-11-22 17:04 +0000`).
- **Objetivo y en alcance:** `172.17.0.0/24` (en alcance). Identificar equipos activos **sin escanear puertos**.
- **Qué se hizo:** barrido de descubrimiento (ARP/ICMP), **técnica activa no intrusiva**:
  ```bash
  nmap -sn 172.17.0.0/24
  ```
  `-sn` desactiva el escaneo de puertos.
- **Qué salió:** **9 equipos activos** en 4.01 s: `172.17.0.1` (la propia máquina de auditoría, **no es objetivo**) y los **8 objetivos** `172.17.0.3` a `172.17.0.10`. No responden `.2`, `.11`, `.20`, `.30`, `.50`, `.100` ni `.254` (los de la hoja del cliente). Captura: [`03_nmap_sn_hosts_vivos.png`](../evidencias/03_nmap_sn_hosts_vivos.png).
- **Siguiente paso:** puertos TCP y versiones sobre los 8 objetivos con ritmo moderado (`-T3`).

---

## Entrada 03 · Mapeo inicial de puertos TCP y versiones
- **Cuándo:** 2026-11-22 17:06 UTC.
- **Objetivo y en alcance:** `172.17.0.3-10` (en alcance). Levantar puertos y versiones sin intrusión ni fuerza bruta.
- **Qué se hizo:**
  ```bash
  nmap -sV -T3 -oN escaneo_servicios_inicial.txt 172.17.0.3-10
  ```
  `-sV` detecta versión; `-T3` es ritmo normal. **Sin `-p`, solo mira los 1000 puertos TCP más comunes** (limitación corregida en las Entradas 14 y 15).
- **Qué salió** ([`escaneo_servicios_inicial.txt`](../evidencias/salidas-laboratorio/escaneo_servicios_inicial.txt)):
  - **.3:** 21 `vsftpd 2.3.4`; 80 `Apache httpd 2.4.58 (Ubuntu)`.
  - **.4:** 22 `OpenSSH 7.9p1 Debian 10+deb10u4`; 8009 `ajp13`; 8080 `Apache Tomcat 9.0.30`.
  - **.5:** 80 y 443 `Apache httpd 2.4.59 (Debian)`.
  - **.6:** 22 `OpenSSH 9.2p1 Debian 2+deb12u2`; 8080 `Jetty 10.0.18`.
  - **.7:** 22 `OpenSSH 7.7`.
  - **.8:** 22 `OpenSSH 9.6p1 Ubuntu 3ubuntu13.5`; 80 `Apache httpd 2.4.58`; 139 y 445 `Samba smbd 4`.
  - **.9:** 22 `OpenSSH 9.2p1 Debian 2+deb12u10`; 80 `Apache httpd 2.4.68`.
  - **.10:** 80 `SimpleHTTPServer 0.6 (Python 3.11.2)`.
- **Siguiente paso:** escaneo UDP y **mapa TCP completo (`-p-`)**. El `-p-` se hizo en las Entradas 14 y 15.

---

## Entrada 04 · Escaneo UDP (20 puertos)
- **Cuándo:** 2026-11-22 17:08 UTC.
- **Objetivo y en alcance:** `172.17.0.3-10` (en alcance). Ver qué servicios hablan UDP.
- **Qué se hizo:**
  ```bash
  nmap -sU --top-ports 20 -T3 -oN escaneo_udp.txt 172.17.0.3-10
  ```
- **Qué salió** ([`escaneo_udp.txt`](../evidencias/salidas-laboratorio/escaneo_udp.txt)):
  - **.8:** `137/udp open` (`netbios-ns`) y `138/udp open|filtered`.
  - **.3** y **.7:** varios puertos `open|filtered`. **`open|filtered` significa "no se sabe": no es abierto.**
  - El resto: `closed`.
  - **Limitación:** solo se miraron 20 puertos; el 623 (IPMI) **no estaba** entre ellos.
- **Siguiente paso:** repetir con más puertos y ritmo limitado (Entrada 18) y mirar el 623 (Entrada 12). Los `open|filtered` de `.3` y `.7` resultaron ser ruido (Entrada 18).

---

## Entrada 05 · Enumeración anónima de Samba (172.17.0.8)
- **Cuándo:** 2026-11-22 17:09:48 UTC (inicio de `enum4linux`; terminó a las 17:10:11).
- **Objetivo y en alcance:** `172.17.0.8` (en alcance). Recursos compartidos, grupo de trabajo y cuentas **mediante sesión nula**, sin probar contraseñas.
- **Qué se hizo:**
  ```bash
  enum4linux -a -u "" -p "" 172.17.0.8 > enum_samba_172.17.0.8.txt
  head -n 45 enum_samba_172.17.0.8.txt
  grep -E "user:|Users|sharename|Disk|Mapping" enum_samba_172.17.0.8.txt -A 2 -B 1
  tail -n 60 enum_samba_172.17.0.8.txt
  ```
  `-u "" -p ""` usa credenciales **vacías**. Los tres últimos comandos solo **leen** el fichero local.
- **Qué salió** ([`enum_samba_172.17.0.8.txt`](../evidencias/salidas-laboratorio/enum_samba_172.17.0.8.txt), captura [`04_enum4linux_samba.jpg`](../evidencias/04_enum4linux_samba.jpg)):
  - `Server 172.17.0.8 allows sessions using username '', password ''` (sesión nula permitida).
  - Grupo de trabajo `ESEEMEB.DL`, NetBIOS `SAMBASERVER`.
  - Cuentas: `usuario1` (RID 1000), `usuario2` (1001), `usuario3` (1002), `satriani7` (1003), `administrador` (1004), `nobody` (501) y el usuario Unix `ubuntu` (UID 1000).
  - Recursos: `myshare` (listado correcto), `backup24` y `home` (denegados), `IPC$`.
- **Siguiente paso:** listar `myshare` de forma anónima.

---

## Entrada 06 · Listado del recurso público `myshare`
- **Cuándo:** 2026-11-22 17:12 UTC (el log no recoge un `date` propio; hora aproximada entre 17:10:11 y 17:13).
- **Objetivo y en alcance:** `172.17.0.8` (en alcance). Ver qué hay en el recurso de acceso anónimo.
- **Qué se hizo:**
  ```bash
  smbclient -N //172.17.0.8/myshare -c "ls"
  ```
  `-N` indica que no se use contraseña.
- **Qué salió:** `Anonymous login successful`; un fichero, `access.txt` (956 bytes, 6 oct 2024).
- **Siguiente paso:** descargar y leer `access.txt`.

---

## Entrada 07 · Fuga de un token JWT en `access.txt`
- **Cuándo:** 2026-11-22 17:13 UTC (sin `date` propio; hora aproximada).
- **Objetivo y en alcance:** `172.17.0.8` (en alcance).
- **Qué se hizo:**
  ```bash
  smbclient -N //172.17.0.8/myshare -c "get access.txt"
  cat access.txt
  ```
- **Qué salió:** el fichero contiene un **token JWT firmado RS256** (`eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.<payload>.<firma omitida>`). **La firma no se reproduce en este cuaderno y está redactada en el log publicado** (ver la nota de evidencias del README). La decodificación del contenido se ejecutó y registró en la **Entrada 17**.
- **Siguiente paso:** decodificar el payload (Entrada 17) y pasar a la enumeración web.

---

## Entrada 08 · Enumeración web: cabeceras y títulos
- **Cuándo:** 2026-11-22 17:14:00 UTC (cabecera `Date:` de las respuestas HTTP).
- **Objetivo y en alcance:** `172.17.0.3-10` (en alcance). Identificar las aplicaciones web **solo con cabeceras y título**.
- **Qué se hizo:** peticiones `GET`/`HEAD` (técnica activa no intrusiva), con tiempo máximo de 2 s:
  ```bash
  for ip in 172.17.0.3 172.17.0.4:8080 172.17.0.5 172.17.0.6:8080 172.17.0.8 172.17.0.9 172.17.0.10; do
      echo "=== PROBANDO $ip ==="
      curl -s -I -m 2 http://$ip/ | head -n 8
      curl -s -m 2 http://$ip/ | grep -i "<title>"
  done
  ```
- **Qué salió:** `.3` Apache 2.4.58 (página por defecto de Ubuntu); `.4:8080` **Apache Tomcat/9.0.30**; `.5` Apache 2.4.59 (página por defecto de Debian); `.6:8080` **HTTP 403** con `X-Jenkins: 2.441` y `X-Hudson: 1.395`; `.8` `<title>Login</title>`; `.9` `test page`; `.10` `SimpleHTTP/0.6 Python/3.11.2` con título `CTG · IPMI bajo exposición`.
- **Siguiente paso:** inspeccionar el formulario de `.8` y el contenido de `.10`.

---

## Entrada 09 · Formulario de login de `.8` y cabecera de la página de `.10`
- **Cuándo:** 2026-11-22 17:15 UTC (sin `date` propio; hora aproximada).
- **Objetivo y en alcance:** `172.17.0.8` y `172.17.0.10` (en alcance). Mirar el formulario **sin enviarle datos**.
- **Qué se hizo:**
  ```bash
  curl -s http://172.17.0.8/ | grep -i -E "form|input|action|name"
  curl -s http://172.17.0.10/ | grep -i -E "h1|h2|h3|p" | head -n 15
  ```
- **Qué salió:**
  - **.8:** formulario `POST` con campos `email` y `password`. No se envió nada.
  - **.10:** el filtro devolvió **solo CSS y el título**, no el texto de la página. **No se obtuvo contenido útil**; la página se leyó correctamente en la Entrada 10.
- **Siguiente paso:** leer completa la página de `.10` (Entrada 10).

---

## Entrada 10 · Lectura completa de la página de `172.17.0.10`
- **Cuándo:** 2026-11-22 17:58:44 UTC (`date` de la sesión 2, seguido de los comandos).
- **Objetivo y en alcance:** `172.17.0.10` (en alcance). Leer qué dice la página (petición web de solo lectura).
- **Qué se hizo:**
  ```bash
  date
  curl -s http://172.17.0.10/ -o ipmi_172.17.0.10.html
  sed -n '/<body/,$p' ipmi_172.17.0.10.html | sed 's/<[^>]*>//g' | grep -v '^\s*$'
  ```
  `-o` guarda la página; los `sed` descartan el CSS y las etiquetas.
- **Qué salió** ([`ipmi_172.17.0.10.html`](../evidencias/salidas-laboratorio/ipmi_172.17.0.10.html), 10 398 bytes, el mismo tamaño que el `Content-Length` de la Entrada 08): documento técnico sobre **IPMI/BMC**: menciona el **puerto UDP 623**, **SOL** (consola serie por red), la mala práctica de exponer la interfaz de gestión en una red amplia y el **CVE-2013-4786** (fallo del intercambio RAKP de IPMI 2.0). **Es documentación, no evidencia de que el servicio exista.**
- **Siguiente paso:** comprobar si **UDP 623** está abierto (Entrada 12), tras verificar la categoría del script (Entrada 11).

---

## Entrada 11 · Verificación de la categoría del script `ipmi-version`
- **Cuándo:** 2026-11-22 18:01:58 UTC (`date`) y 18:02 UTC (cabecera de Nmap).
- **Objetivo y en alcance:** no genera tráfico hacia la red; consulta la ayuda local de Nmap. Comprobar antes de usarlo que **no es de una categoría prohibida**.
- **Qué se hizo:**
  ```bash
  script -a registro_sesion_terminal_2.log
  date
  nmap --script-help ipmi-version
  ```
- **Qué salió:** `Categories: discovery safe` (ninguna de `exploit`, `intrusive`, `dos`, `brute` ni `malware`). Descripción: "Performs IPMI Information Discovery through Channel Auth probes".
- **Siguiente paso:** comprobar el estado de UDP 623 en los 8 equipos.

---

## Entrada 12 · Estado de UDP 623 en los 8 equipos
- **Cuándo:** 2026-11-22 18:03 UTC.
- **Objetivo y en alcance:** `172.17.0.3-10` (en alcance), **solo el puerto 623/udp**, con ritmo suave por si algún equipo es delicado.
- **Qué se hizo:**
  ```bash
  nmap -sU -p 623 -T2 -oN escaneo_udp_623.txt 172.17.0.3-10
  ```
- **Qué salió** ([`escaneo_udp_623.txt`](../evidencias/salidas-laboratorio/escaneo_udp_623.txt), 2.42 s): **`172.17.0.10`: `623/udp open asf-rmcp`**; los otros 7 equipos: `closed`.
- **Siguiente paso:** identificar la versión de IPMI en `.10` con `ipmi-version`.

---

## Entrada 13 · Versión de IPMI en `172.17.0.10`
- **Cuándo:** 2026-11-22 18:04 UTC.
- **Objetivo y en alcance:** `172.17.0.10`, puerto `623/udp` (en alcance). Script `discovery safe` verificado en la Entrada 11; **no inicia sesión ni intercambia credenciales**.
- **Qué se hizo:**
  ```bash
  nmap -sU -p 623 -T2 --script ipmi-version -oN ipmi_version_172.17.0.10.txt 172.17.0.10
  ```
- **Qué salió** ([`ipmi_version_172.17.0.10.txt`](../evidencias/salidas-laboratorio/ipmi_version_172.17.0.10.txt), 1.47 s):
  ```text
  ipmi-version:
    Version: IPMI-2.0
    UserAuth: password, md5, md2, null
    PassAuth: auth_msg, auth_user, non_null_user
    Level: 1.5, 2.0
  ```
  Se registra la salida **literal**; el valor `null` no se interpreta sin confirmarlo en la documentación del script.
- **Siguiente paso:** mapa TCP completo de `.10` con ritmo limitado (Entrada 14). **No se comprueba CVE-2013-4786**: exigiría el intercambio RAKP, que interactúa con la vulnerabilidad (prohibido).

---

## Entrada 14 · Mapa TCP completo de `172.17.0.10`
- **Cuándo:** 2026-11-22 18:06 UTC.
- **Objetivo y en alcance:** `172.17.0.10` (en alcance). Los 65 535 puertos TCP, **más despacio** por ser probablemente el equipo delicado (BMC).
- **Qué se hizo:**
  ```bash
  nmap -p- -T3 --max-rate 100 -oN tcp_completo_172.17.0.10.txt 172.17.0.10
  ```
  `--max-rate 100` limita a 100 paquetes por segundo. Sin `-sV` (solo descubrir puertos).
- **Qué salió** ([`tcp_completo_172.17.0.10.txt`](../evidencias/salidas-laboratorio/tcp_completo_172.17.0.10.txt), 656.01 s): `65534 closed`; **solo `80/tcp open`**.
- **Siguiente paso:** mapa TCP completo de `.3` a `.9`.

---

## Entrada 15 · Mapa TCP completo de `172.17.0.3-9`
- **Cuándo:** 2026-11-22 18:17 UTC.
- **Objetivo y en alcance:** `172.17.0.3-9` (en alcance).
- **Qué se hizo:**
  ```bash
  nmap -p- -T3 --max-rate 300 -oN tcp_completo_172.17.0.3-9.txt 172.17.0.3-9
  ```
- **Qué salió** ([`tcp_completo_172.17.0.3-9.txt`](../evidencias/salidas-laboratorio/tcp_completo_172.17.0.3-9.txt), 1530.82 s):
  - **.3:** 21, 80. **.4:** 22, 8009, 8080. **.6:** 22, 8080. **.7:** 22. **.8:** 22, 80, 139, 445. **.9:** 22, 80. (Coinciden con la Entrada 03.)
  - **.5:** 80, 443 **y 7 puertos nuevos: 1883, 5672, 8161, 46335, 61613, 61614, 61616.** Estos puertos **no aparecían** en la Entrada 03.
- **Siguiente paso:** identificar servicio y versión de los 7 puertos nuevos de `.5`.

---

## Entrada 16 · Servicios en los puertos altos de `172.17.0.5`
- **Cuándo:** 2026-11-22 18:44 UTC (cabecera de Nmap) y entre 18:44 y 18:47 (el `curl`, sin `date` propio).
- **Objetivo y en alcance:** `172.17.0.5`, **solo los 7 puertos nuevos** (en alcance). Sin scripts de Nmap.
- **Qué se hizo:**
  ```bash
  nmap -sV -T3 -p 1883,5672,8161,46335,61613,61614,61616 -oN servicios_172.17.0.5_altos.txt 172.17.0.5
  curl -s -I -m 5 http://172.17.0.5:8161/
  ```
  `curl -I` pide solo las cabeceras. **No se probaron credenciales.**
- **Qué salió** ([`servicios_172.17.0.5_altos.txt`](../evidencias/salidas-laboratorio/servicios_172.17.0.5_altos.txt), 31.85 s):
  ```text
  1883/tcp  mqtt
  5672/tcp  amqp?
  8161/tcp  http       Jetty 9.4.39.v20210325
  46335/tcp tcpwrapped
  61613/tcp stomp      Apache ActiveMQ
  61614/tcp http       Jetty 9.4.39.v20210325
  61616/tcp apachemq   ActiveMQ OpenWire transport 5.15.15
  ```
  - `curl`: `HTTP/1.1 401 Unauthorized` con `WWW-Authenticate: basic realm="ActiveMQRealm"`.
  - Ante la sonda de Nmap, `61613` devolvió un error STOMP con traza Java (`org.apache.activemq…`).
  - `46335` acepta la conexión y la cierra (`tcpwrapped`): **no identificado**.
- **Siguiente paso:** contrastar `ActiveMQ 5.15.15` con NVD, EPSS y KEV (Entrada 20).

---

## Entrada 17 · Comprobaciones locales: Samba y token JWT
- **Cuándo:** 2026-11-22 18:47:53 / 18:48:06 / 18:48:18 UTC (`date` antes de cada comando).
- **Objetivo y en alcance:** ficheros **locales** del Kali (`enum_samba_172.17.0.8.txt`, `access.txt`). **No generan tráfico de red.**
- **Qué se hizo:**
  ```bash
  grep -A8 "OS information" enum_samba_172.17.0.8.txt
  cut -d. -f2 access.txt | tr '_-' '/+' | base64 -d 2>/dev/null; echo
  date -u -d @1728163973
  ```
  El 2.º extrae la parte central (*payload*) del JWT y la decodifica de base64; el 3.º convierte la caducidad a hora legible.
- **Qué salió:**
  - **Samba:** `srvinfo` solo dice `SAMBASERVER Wk Sv PrQ Unx NT SNT EseEmeB Samba Server`; **no hay versión exacta**.
  - **Payload del JWT:** `{"email":"satriani7@eseemeb.dl","role":"user","iat":1728160373,"exp":1728163973,"jwk":{"kty":"RSA","n":"…","e":65537}}` (clave pública RSA incrustada; módulo omitido aquí).
  - **Caducidad:** `Sat Oct 5 09:32:53 PM UTC 2024`. Emitido a las 20:32:53, **caducó hace más de dos años**.
- **Siguiente paso:** ampliar el UDP (Entrada 18). El token **no se usa**.

---

## Entrada 18 · UDP ampliado (100 puertos)
- **Cuándo:** 2026-11-22 18:48 UTC.
- **Objetivo y en alcance:** `172.17.0.3-10` (en alcance).
- **Qué se hizo:**
  ```bash
  nmap -sU --top-ports 100 -T3 --max-rate 100 -oN udp_top100.txt 172.17.0.3-10
  ```
- **Qué salió** ([`udp_top100.txt`](../evidencias/salidas-laboratorio/udp_top100.txt), 107.41 s, `8 hosts up`):
  - Solo dos servicios UDP abiertos: **`.8` `137/udp netbios-ns`** y **`.10` `623/udp asf-rmcp`**.
  - **`.3` y `.7` salen con los 100 puertos `closed`.** Los `open|filtered` de la Entrada 04 eran **ruido** del escaneo (límite de mensajes ICMP), no puertos abiertos.
  - **`.8`:** 45 puertos `open|filtered` (inconcluso).
  - `8 hosts up` tras todos los escaneos: **ningún equipo se cayó**.
- **Siguiente paso:** intentar la versión de Samba (Entrada 19).

---

## Entrada 19 · Versión de Samba con `smb-os-discovery`
- **Cuándo:** 2026-11-22 18:53 UTC (`date`: 18:53:37).
- **Objetivo y en alcance:** `172.17.0.8:445` (en alcance). Script `safe` verificado antes; usa **la sesión anónima** (la misma de la Entrada 05), sin probar credenciales.
- **Qué se hizo:**
  ```bash
  nmap --script-help smb-os-discovery
  date
  nmap -p445 -T3 --script smb-os-discovery -oN smb_os_discovery_172.17.0.8.txt 172.17.0.8
  ```
- **Qué salió:** `Categories: default discovery safe`. El escaneo (0.71 s) **no devolvió información de SO ni de versión** (Samba "blanquea" esos campos). Resultado en [`smb_os_discovery_172.17.0.8.txt`](../evidencias/salidas-laboratorio/smb_os_discovery_172.17.0.8.txt).
- **Conclusión:** la **versión exacta de Samba no se pudo determinar** sin autenticarse; SambaCry (CVE-2017-7494) queda **no confirmado y no descartado**. No se insiste con más sondeos.
- **Siguiente paso:** cerrar el laboratorio (`exit`), copiar las salidas a `evidencias/` **excluyendo `access.txt`** y redactar los entregables.

---

## Entrada 20 · Consulta de fuentes públicas (CVSS, EPSS, KEV)
- **Cuándo:** 2026-11-22 (entre las sesiones de laboratorio y la redacción de los entregables; las peticiones no llevan hora propia: el EPSS indica la fecha `2026-11-22`).
- **Objetivo y en alcance:** **no aplica**: son consultas a **fuentes públicas** desde el ordenador de la auditora (Windows), con ayuda del asistente Claude Code. **No tocan la red de Gadir.** Solo viajan identificadores de CVE.
- **Qué se hizo:** peticiones HTTP de lectura a:
  ```text
  https://api.first.org/data/v1/epss?cve=<lista de CVE>                  (EPSS y percentil)
  https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json  (KEV, catálogo 2026.11.20)
  https://services.nvd.nist.gov/rest/json/cves/2.0?cveId=<CVE>            (CVSS, CWE, rangos de versiones)
  https://security-tracker.debian.org/tracker/<CVE>                       (versión corregida por Debian)
  ```
- **Qué salió:** las cifras de EPSS, KEV, CVSS y CWE de [`registro-vulns.md`](../registro-vulns.md), que sustituyen a los valores aproximados de borradores anteriores. Hallazgos: el CWE de Jenkins, vsftpd y OpenSSH difería de lo anotado antes; el CVE-2026-123158 ("sshole") **sí** consta en KEV (2026-11-09).
- **Siguiente paso:** priorizar y redactar `registro-vulns.md` e `informe-cliente.md`.

---

## Matriz de trazabilidad (dato del informe → comprobación → evidencia)

| Dato afirmado en los entregables | Entrada | Evidencia |
|---|---|---|
| 8 equipos vivos (`.3` a `.10`); `.1` es la máquina de auditoría | 01, 02 | [registro_sesion_terminal.log](../evidencias/registro_sesion_terminal.log), [03_nmap_sn_hosts_vivos.png](../evidencias/03_nmap_sn_hosts_vivos.png) |
| No responden `.2`, `.11`, `.20`, `.30`, `.50`, `.100`, `.254` | 02 | ídem |
| Mapa TCP completo (65 535 puertos) y puertos ocultos de `.5` | 14, 15 | [tcp_completo_172.17.0.10.txt](../evidencias/salidas-laboratorio/tcp_completo_172.17.0.10.txt), [tcp_completo_172.17.0.3-9.txt](../evidencias/salidas-laboratorio/tcp_completo_172.17.0.3-9.txt) |
| Versiones de vsftpd, Tomcat, Jetty, Samba y las 5 versiones de OpenSSH | 03 | [escaneo_servicios_inicial.txt](../evidencias/salidas-laboratorio/escaneo_servicios_inicial.txt) |
| ActiveMQ OpenWire 5.15.15 y consola `8161` con `401` | 16 | [servicios_172.17.0.5_altos.txt](../evidencias/salidas-laboratorio/servicios_172.17.0.5_altos.txt) |
| Jenkins 2.441 con respuesta 403; Tomcat 9.0.30 | 08 | [registro_sesion_terminal.log](../evidencias/registro_sesion_terminal.log) |
| IPMI: `623/udp` abierto solo en `.10` | 12, 18 | [escaneo_udp_623.txt](../evidencias/salidas-laboratorio/escaneo_udp_623.txt), [udp_top100.txt](../evidencias/salidas-laboratorio/udp_top100.txt) |
| IPMI 2.0 (salida literal de `ipmi-version`) | 11, 13 | [ipmi_version_172.17.0.10.txt](../evidencias/salidas-laboratorio/ipmi_version_172.17.0.10.txt) |
| Contenido de la página de `.10` | 10 | [ipmi_172.17.0.10.html](../evidencias/salidas-laboratorio/ipmi_172.17.0.10.html) |
| Sesión nula, grupo `ESEEMEB.DL`, recursos y cuentas de `.8` | 05 | [enum_samba_172.17.0.8.txt](../evidencias/salidas-laboratorio/enum_samba_172.17.0.8.txt), [04_enum4linux_samba.jpg](../evidencias/04_enum4linux_samba.jpg) |
| `access.txt` con un JWT; caducó el 2024-10-05 21:32:53 UTC | 06, 07, 17 | [registro_sesion_terminal.log](../evidencias/registro_sesion_terminal.log), [registro_sesion_terminal_2.log](../evidencias/salidas-laboratorio/registro_sesion_terminal_2.log) |
| Versión de Samba no determinada | 17, 19 | [smb_os_discovery_172.17.0.8.txt](../evidencias/salidas-laboratorio/smb_os_discovery_172.17.0.8.txt) |
| Solo hay dos servicios UDP abiertos (`137` en `.8` y `623` en `.10`); los `open|filtered` de `.3` y `.7` eran ruido | 04, 18 | [escaneo_udp.txt](../evidencias/salidas-laboratorio/escaneo_udp.txt), [udp_top100.txt](../evidencias/salidas-laboratorio/udp_top100.txt) |
| Ningún equipo se cayó durante los escaneos (`8 hosts up` al final) | 18 | [udp_top100.txt](../evidencias/salidas-laboratorio/udp_top100.txt) |
| CVSS, EPSS, KEV, CWE y rangos de versiones | 20 | consultas a NVD, FIRST y CISA con fecha 2026-11-22 |

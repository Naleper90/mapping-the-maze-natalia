# Inventario de Equipos, Servicios y Cuentas de Red

- **Cliente:** Gadir Logística, S.L. (Puerto de Cádiz)
- **Alcance legal:** `172.17.0.0/24` (definido en [`scope.txt`](scope.txt))
- **Auditora:** Natalia Alejo Pérez
- **Fecha de auditoría:** 22/11/2026
- **Trazabilidad:** cada dato remite a una entrada del [cuaderno](cuadernos/natalia-alejo.md) y a un fichero de [evidencias](evidencias/).

---

## 1. Cómo se ha obtenido (técnicas activas no intrusivas)

| Qué | Comando | Entrada | Fichero |
|---|---|---|---|
| Equipos vivos | `nmap -sn 172.17.0.0/24` | 02 | [registro_sesion_terminal.log](evidencias/registro_sesion_terminal.log) |
| Puertos TCP **completos** (1-65535) | `nmap -p- -T3 --max-rate 100` (`.10`) y `--max-rate 300` (`.3-.9`) | 14, 15 | [tcp_completo_172.17.0.10.txt](evidencias/salidas-laboratorio/tcp_completo_172.17.0.10.txt), [tcp_completo_172.17.0.3-9.txt](evidencias/salidas-laboratorio/tcp_completo_172.17.0.3-9.txt) |
| Servicios y versiones TCP | `nmap -sV -T3` (1000 puertos comunes) y `-sV` sobre los puertos altos de `.5` | 03, 16 | [escaneo_servicios_inicial.txt](evidencias/salidas-laboratorio/escaneo_servicios_inicial.txt), [servicios_172.17.0.5_altos.txt](evidencias/salidas-laboratorio/servicios_172.17.0.5_altos.txt) |
| UDP | `nmap -sU --top-ports 20` y `--top-ports 100 --max-rate 100`; puerto 623 aparte | 04, 12, 18 | [escaneo_udp.txt](evidencias/salidas-laboratorio/escaneo_udp.txt), [udp_top100.txt](evidencias/salidas-laboratorio/udp_top100.txt), [escaneo_udp_623.txt](evidencias/salidas-laboratorio/escaneo_udp_623.txt) |

**Notas de método**
- **Se ha encontrado más de lo que mostraba el primer escaneo.** `nmap -sV` solo mira los 1000 puertos TCP más comunes. El mapa completo reveló **7 puertos ocultos en `172.17.0.5`** (mensajería ActiveMQ). El UDP por defecto (20 puertos) no incluía el **623**, que está abierto en `172.17.0.10`.
- **Cuidado con los equipos delicados.** No se ha identificado cuál es el "equipo antiguo y delicado" del enunciado, así que todos recibieron el mismo trato: `-T3` con límite de paquetes por segundo (`--max-rate`) y `-T2` para el 623. Tras todos los escaneos, el barrido de las 18:48 siguió viendo `8 hosts up`: **ningún equipo se cayó**.
- **No se ejecutó detección de SO (`-O`).** El sistema operativo se deduce de los **banners** (sufijos como `Debian`, `Ubuntu`, `deb10u4`…) y se indica en la tabla cómo se deduce.
- **No se usó ningún script de Nmap salvo `ipmi-version` y `smb-os-discovery`** (ambos categoría `safe`, verificada en cada caso).
- **El host `172.17.0.1` es la propia máquina de auditoría** (`docker0`). Nmap lo cuenta como el noveno equipo vivo, pero no es un objetivo. Por eso son **8** equipos.

---

## 2. Inventario real descubierto (8 equipos vivos)

| IP | SO (cómo se deduce) | Puertos TCP abiertos | Puertos UDP abiertos | Servicio y versión exacta | Función probable |
|---|---|---|---|---|---|
| **172.17.0.3** | Linux Ubuntu (banner `Apache … (Ubuntu)`) | 21, 80 | ninguno | FTP **vsftpd 2.3.4**; HTTP **Apache httpd 2.4.58 (Ubuntu)** | Servidor FTP y web por defecto |
| **172.17.0.4** | Linux Debian 10 (`OpenSSH 7.9p1 Debian 10+deb10u4`) | 22, 8009, 8080 | ninguno | SSH **OpenSSH 7.9p1 Debian 10+deb10u4**; **AJP13** Apache Jserv 1.3; HTTP **Apache Tomcat 9.0.30** | Servidor de aplicaciones Java |
| **172.17.0.5** | Linux Debian (banner `Apache … (Debian)`) | 80, 443, **1883, 5672, 8161, 46335, 61613, 61614, 61616** | ninguno | HTTP/HTTPS **Apache httpd 2.4.59 (Debian)**; **MQTT** (1883); **AMQP** (5672, `amqp?`); **Jetty 9.4.39.v20210325** (8161, consola web ActiveMQ); `tcpwrapped` (46335); **STOMP** Apache ActiveMQ (61613); **Jetty 9.4.39** (61614); **ActiveMQ OpenWire 5.15.15** (61616) | **Broker de mensajería Apache ActiveMQ** (probable "mensajería de los operadores") |
| **172.17.0.6** | Linux Debian 12 (`OpenSSH 9.2p1 Debian 2+deb12u2`) | 22, 8080 | ninguno | SSH **OpenSSH 9.2p1 Debian 2+deb12u2**; HTTP **Jetty 10.0.18** con **Jenkins 2.441** (cabecera `X-Jenkins`) | Servidor de integración continua |
| **172.17.0.7** | Linux, **distribución no determinada** (OpenSSH sin sufijo de distribución) | 22 | ninguno | SSH **OpenSSH 7.7** (protocolo 2.0) | Servidor con SSH desactualizado |
| **172.17.0.8** | Linux Ubuntu (`OpenSSH 9.6p1 Ubuntu 3ubuntu13.5`) | 22, 80, 139, 445 | **137** (`netbios-ns`) | SSH **OpenSSH 9.6p1 Ubuntu 3ubuntu13.5**; HTTP **Apache httpd 2.4.58** (portal de login); **Samba smbd 4** (versión exacta no determinada), NetBIOS `SAMBASERVER` | Servidor de ficheros y portal |
| **172.17.0.9** | Linux Debian 12 (`OpenSSH 9.2p1 Debian 2+deb12u10`) | 22, 80 | ninguno | SSH **OpenSSH 9.2p1 Debian 2+deb12u10**; HTTP **Apache httpd 2.4.68 (Debian)** | Servidor web de pruebas |
| **172.17.0.10** | Linux probable (`SimpleHTTP/0.6 Python/3.11.2`), no determinado | 80 | **623** (`asf-rmcp`, IPMI 2.0) | HTTP **SimpleHTTP/0.6 Python 3.11.2**; **IPMI 2.0** sobre RMCP (UDP) | Interfaz de gestión fuera de banda (BMC) y página informativa |

**Correcciones respecto al primer borrador (ya no se afirma):**
- Los puertos UDP `open|filtered` de `.3` (53, 67, 68, 69, 123, 137, 445, 631, 1434) y de `.7` (53, 67, 137-139, 500, 520, 4500, 49152) del escaneo de las 17:08 **no eran abiertos**: con el ritmo limitado del escaneo de las 18:48 aparecen todos **cerrados**. Eran ruido del escaneo (límite de mensajes ICMP). **No hay servicios VPN en `.7`.**
- En `.8`, `138/udp` y otros 44 puertos siguen `open|filtered` (inconcluso); solo `137/udp` está confirmado `open`.
- "Mapeo completo de puertos TCP" **ya es cierto**: se ha usado `-p-` en los 8 equipos.

---

## 3. Qué deja ver cada servicio sin autenticarse (enumeración)

> Regla del cliente: no se prueban credenciales ni se accede a lo que exija autenticación. Cada fila indica **qué se obtuvo, cómo y qué límite hubo**.

| Equipo · servicio | Qué se obtuvo y cómo | Limitaciones |
|---|---|---|
| `.3` FTP 21 | Banner `vsftpd 2.3.4` (`nmap -sV`). | No se probó el acceso anónimo (sería probar un login por defecto). No se pidió nada al servicio. |
| `.3` HTTP 80 | Cabeceras y título: `Apache2 Ubuntu Default Page: It works` (Entrada 08). | Solo cabeceras y título; no se enumeraron directorios. |
| `.4` HTTP 8080 | `Apache Tomcat/9.0.30`, página estándar (Entrada 08). | No se pidió `/manager` ni se probaron credenciales. |
| `.4` AJP 8009 | Servicio `ajp13 Apache Jserv 1.3` (Entradas 03 y 15). | No se enviaron paquetes AJP. |
| `.5` HTTP/HTTPS 80/443 | Página `Apache2 Debian Default Page: It works` en el 80 (Entrada 08). | No se inspeccionó el certificado del 443. |
| `.5` ActiveMQ | `61616` OpenWire **5.15.15**; `61613` STOMP (el servicio devolvió un error con traza Java de `org.apache.activemq…` ante la sonda de `nmap`); `8161` y `61614` Jetty 9.4.39; `curl -I :8161` → `401` `realm="ActiveMQRealm"` (Entrada 16). | `5672` (AMQP) y `1883` (MQTT) sin versión; `46335` es `tcpwrapped` (acepta la conexión y la cierra), **no identificado**. Consola `8161` protegida: no se probaron credenciales. |
| `.6` Jenkins 8080 | Cabeceras `X-Jenkins: 2.441` y `X-Hudson: 1.395`; la raíz devuelve **403** (Entrada 08). | Sin acceso anónimo no se enumera nada más. |
| `.7` SSH 22 | Banner `OpenSSH 7.7`. | Es el único puerto abierto: no hay más superficie. La distribución no se pudo determinar. |
| `.8` Samba 139/445 | **Sesión nula** permitida; grupo `ESEEMEB.DL`, NetBIOS `SAMBASERVER`; recursos `myshare` (listado anónimo correcto), `backup24` y `home` (denegados); **cuentas de usuario** (ver sección 4) (Entrada 05). | Versión exacta de Samba **no determinada**: `srvinfo` solo dice "Samba Server" y `smb-os-discovery` no devolvió datos (Entrada 19). |
| `.8` HTTP 80 | Portal con título `Login` y formulario `POST` con campos `email` y `password` (Entrada 09). | No se envió ningún dato al formulario. |
| `.8` NetBIOS 137/udp | Abierto (`netbios-ns`). | `138/udp` inconcluso (`open|filtered`). |
| `.9` HTTP 80 | `test page`, Apache 2.4.68 (Entrada 08). | Solo cabeceras y título. |
| `.10` HTTP 80 | Documentación técnica sobre IPMI/BMC (UDP 623, SOL, CVE-2013-4786) (Entrada 10). | Es una página informativa, **no** evidencia del servicio. |
| `.10` IPMI 623/udp | Abierto; `ipmi-version`: `IPMI-2.0`, niveles `1.5, 2.0`, tipos de autenticación `password, md5, md2, null` (Entrada 13). | No se interpreta el valor `null`; no se inició ninguna sesión IPMI. |

---

## 4. Cuentas de usuario y recursos descubiertos sin autenticar

### 4.1. Cuentas (obtenidas de `172.17.0.8` por sesión nula y *RID cycling*)
- `usuario1` (RID 1000 / `0x3e8`), `usuario2` (1001), `usuario3` (1002)
- `satriani7` (RID 1003 / `0x3eb`): cuenta vinculada al correo `satriani7@eseemeb.dl` según el token.
- `administrador` (RID 1004 / `0x3ec`): **el nombre sugiere cuenta privilegiada, pero no hay evidencia de sus permisos.**
- `nobody` (RID 501).
- `ubuntu`: usuario Unix, **UID 1000** (SID `S-1-22-1-1000`); los otros cinco nombres también existen como usuarios Unix (UID 1001 a 1005).

### 4.2. Recursos compartidos y fuga de secretos
- **`//172.17.0.8/myshare`:** listado y lectura anónimos.
- **Fuga de token JWT (`access.txt`, 956 bytes):** token RS256 del usuario `satriani7@eseemeb.dl` (`role: user`), con una clave pública RSA incrustada (`jwk`). Decodificado en el laboratorio (Entrada 17): emitido `2024-10-05 20:32:53 UTC`, **caducó `2024-10-05 21:32:53 UTC`**. No se ha usado.

---

## 5. Corrección crítica del inventario del cliente (hoja de Juanma)

Se contrasta [`inventario-cliente.csv`](inventario-cliente.csv) con lo descubierto. **La hoja describe sobre todo equipos Windows antiguos que no existen en la red; la red real son 8 equipos Linux.** Regla de redacción: lo que **no responde** al descubrimiento se dice "no responde", no "no existe", porque no se escanearon sus puertos.

### 5.1. Qué sobra
- **`172.17.1.6` (PC-PRUEBAS):** **fuera de alcance** (subred `172.17.1.0/24`). No se ha escaneado ni tocado. Era alcanzable por estar `docker0` en un `/16`, pero el alcance autorizado es solo el `/24`.
- **`172.17.0.2` (FTP-ALMACEN), `.11` (SQL Server), `.20` (srv-mensajeria), `.30` (IMPRESORA-MUELLE), `.50` (NAS-OFICINA), `.100` (srv-backup), `.254` (consola sala técnica):** **no responden** al descubrimiento (`nmap -sn`, 4.01 s, Entrada 02). El mapa TCP completo confirma además que **ningún equipo vivo tiene abiertos** los puertos que la hoja anotaba: 1433 (SQL), 1801 (MSMQ), 9100 (impresora) ni 3389 (RDP).
- **Matiz de `.254` (iLO):** la hoja dice "solo se llega por cable desde la sala", así que podría estar en otra red: *no responde* no es *no existe*.

### 5.2. Qué falta (equipos vivos no inventariados)
- **`.3`:** vsftpd 2.3.4 y Apache 2.4.58.
- **`.4`:** **Tomcat 9.0.30 y AJP13** (8009).
- **`.5`:** **broker ActiveMQ 5.15.15** (7 puertos ocultos al escaneo por defecto) y Apache 2.4.59 con HTTPS.
- **`.6`:** **Jenkins 2.441**.
- **`.7`:** servidor con **OpenSSH 7.7**.
- **`.8`:** servidor **Samba** con un portal de login.
- **`.10` ya figuraba en la hoja** (con datos erróneos), pero **la consola IPMI (UDP 623) no constaba en ningún sitio**.

### 5.3. Qué está mal
- **`172.17.0.10`** (hoja: *Windows Server 2008 R2, IIS 7.5, puertos 80/443*): **falso.** Es un host con Python `SimpleHTTP/0.6` que solo tiene abierto el **80/tcp** (confirmado con los 65 535 puertos) y expone **IPMI 2.0 en UDP 623**.
- **`172.17.0.9`** (hoja: *TPV-RECEPCION, Windows 7, RDP 3389*): **falso.** Es **Debian** con Apache 2.4.68 y OpenSSH 9.2p1; no tiene 3389 abierto.
- **Hipótesis de equivalencias** (pendientes de confirmar con el cliente, no afirmadas): la "mensajería" (`.20`, MSMQ en Windows Server 2003) sería el broker **ActiveMQ de `.5`**; el NAS (`.50`) y su carpeta compartida serían el Samba de **`.8`**; el FTP de Windows XP (`.2`) sería el vsftpd de **`.3`**; la consola de gestión (`.254`, iLO 2) sería el **BMC IPMI de `.10`**.
- **`.11`:** la hoja duda si es la misma máquina que el portal (`.10`). **No lo es:** no hay equipo en `.11` y `.10` no es el portal Windows.

---

## 6. Limitaciones generales
- **SO por banner** (no `-O`). Para `.7` y `.10` no se pudo determinar con certeza.
- **Versión de Samba** no determinada; **versión y función de `46335/tcp`** (`.5`) no identificadas.
- **UDP:** se cubrieron los 100 puertos más comunes, el 623 y la comprobación de `137/udp`. No es un barrido exhaustivo de los 65 535 puertos UDP.
- **Descubrimiento:** equipos que no respondan a ARP/ICMP en el segmento, o que estén en otra red, no aparecerían.

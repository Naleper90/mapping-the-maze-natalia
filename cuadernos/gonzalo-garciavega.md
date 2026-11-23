# Comprobaciones

## Parte 1 · Descubrir y mapear

## Descubrimiento de equipos
- Cuándo: 16/11/2026, 10:03 (hora local; Nmap registró 9:03 UTC)
- Objetivo y por qué está en alcance: 172.17.0.0/24, rango indicado en el enunciado.
- Qué hice: ejecuté `sudo nmap -sn -PR --reason 172.17.0.0/24 -oN descubrimiento.txt` para buscar respuestas ARP, sin escanear puertos.
- Qué salió: Nmap informó respuestas ARP de 172.17.0.3 a 172.17.0.10; 172.17.0.1 aparece como `localhost-response`. La salida final indica 9 hosts activos entre 256 direcciones examinadas.
- Siguiente paso: confirmar que las IP detectadas corresponden a los equipos de mi copia autorizada; después continuar con el mapeo prudente de puertos.
- Evidencia: `descubrimiento.txt`.![nmap](../capturas/gonzalo/parte1/01.png)

## Preparación de la lista de equipos detectados
- Cuándo: 16/11/2026, 10:03 (hora local; Nmap registró 9:03 UTC)
- Objetivo y por qué está en alcance: direcciones obtenidas del descubrimiento de `172.17.0.0/24`, rango indicado en el enunciado.
- Qué hice: extraje las IP detectadas desde `descubrimiento.txt` y preparé `hosts_ips_limpio.txt`, excluyendo `172.17.0.1`, que Nmap marcó como `localhost-response`.
- Qué salió: la lista contiene `172.17.0.3`–`172.17.0.10`.
- Siguiente paso: usar la lista solo para organizar el mapeo de los equipos del laboratorio confirmados por el profesor.
- Evidencia: `descubrimiento.txt`.![descubrimiento](../capturas/gonzalo/parte1/02.png)

## Preparación de lista de IP detectadas
- Cuándo: 16/11/2026, 10:49.
- Objetivo y por qué está en alcance: lista derivada del descubrimiento de `172.17.0.0/24`, el rango indicado en el enunciado.
- Qué hice: filtré `172.17.0.1` de `hosts_ips.txt` y guardé el resultado en `hosts_ips_limpio.txt`. En la captura se ve el comando: `tr ' ' '\n' < hosts_ips.txt | grep -v '^172\.17\.0\.1$' > hosts_ips_limpio.txt`.
- Qué salió: el archivo contiene las IP `172.17.0.3`–`172.17.0.10`. La `.1` se excluyó porque en `descubrimiento.txt` Nmap la marcó como `localhost-response`.
- Siguiente paso: usar la lista para organizar el mapeo de puertos de los equipos confirmados como objetivos del laboratorio.
- Evidencia: `hosts_ips_limpio.txt`
![hostips](../capturas/gonzalo/parte1/03.png)

## Escaneo completo de puertos TCP
- Cuándo: 17/11/2026, 12:24.
- Objetivo y por qué está en alcance: las ocho IP de `hosts_ips_limpio.txt` (`172.17.0.3`–`172.17.0.10`), pertenecientes a la copia de laboratorio confirmada; rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -sS -p- -T3 --reason -iL hosts_ips_limpio.txt -oA tcp-lab`. `-sS` realiza un escaneo SYN de TCP; `-p-` cubre los puertos TCP del 1 al 65535; `-T3` fija el ritmo normal de Nmap; `--reason` muestra el motivo del estado; `-iL` lee los objetivos del archivo. No ejecuté scripts NSE ni comprobaciones de vulnerabilidades.
- Qué salió: Nmap informó que los ocho hosts estaban activos y encontró puertos TCP abiertos. La salida muestra, entre otros, puertos altos como `8009/tcp`, `8161/tcp`, `39223/tcp` y `61613–61616/tcp`. Las etiquetas `SERVICE` son identificaciones preliminares por número de puerto; esta comprobación aún no determina las versiones.
- Siguiente paso: consultar de forma moderada los servicios y versiones de los puertos abiertos; planificar UDP por separado y con cuidado para el equipo delicado.
- Evidencia: archivos `tcp-lab.nmap`, `tcp-lab.gnmap` y `tcp-lab.xml`.
![nmap](../capturas/gonzalo/parte1/04.png)
![nmap](../capturas/gonzalo/parte1/05.png)
![tcp-lab.gnmap](../capturas/gonzalo/parte1/06.png)
![tcp-lab.nmap](../capturas/gonzalo/parte1/07.png)
![tcp-lab.nmap](../capturas/gonzalo/parte1/08.png)
![tcp-lab.xml](../capturas/gonzalo/parte1/09.png)

## Detección del SO
- Cuándo: 19/11/2026,12:29 UTC, horario mi máquina 13:29.
- Objetivo y por qué está en alcance: los ocho equipos activos `172.17.0.3`–`172.17.0.10`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -O --osscan-limit -T3 --reason -oA os-172-17-0-X 172.17.0.X` individualmente contra cada equipo.
- Qué salió: Nmap estimó en los ocho equipos un dispositivo de propósito general con Linux, kernel entre `4.15` y `5.19`. No identificó la distribución ni la versión exacta del sistema operativo.
- Evidencias:`os-172-17-0-3.nmap/.gnmap/.xml` hasta `os-172-17-0-10.nmap/.gnmap/.xml`.
![deteccion](../capturas/gonzalo/parte1/9.1.png)
![deteccion](../capturas/gonzalo/parte1/9.2.png)
![deteccion](../capturas/gonzalo/parte1/9.3.png)
![deteccion](../capturas/gonzalo/parte1/9.4.png)
![deteccion](../capturas/gonzalo/parte1/9.5.png)
![deteccion](../capturas/gonzalo/parte1/9.6.png)
![deteccion](../capturas/gonzalo/parte1/9.7.png)
![deteccion](../capturas/gonzalo/parte1/9.8.png)

## Detección de servicios y versiones — 172.17.0.3
- Cuándo: 17/11/2026, 15:16 (hora local; Nmap registró 14:16 UTC).
- Objetivo y por qué está en alcance: `172.17.0.3`, equipo detectado en `172.17.0.0/24` y confirmado como parte del laboratorio autorizado.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T3 -p 21,80 --reason -oA versiones-172-17-0-3 172.17.0.3`. Consulté únicamente los puertos TCP que el escaneo anterior encontró abiertos; no ejecuté scripts NSE ni pruebas de vulnerabilidad.
- Qué salió: `21/tcp` aparece como FTP, `vsftpd 2.3.4`; `80/tcp` aparece como HTTP, `Apache httpd 2.4.58 (Ubuntu)`. Nmap reporta información de servicio `OS: Unix`.
- Siguiente paso: identificar de la misma forma los servicios y versiones en los puertos abiertos del siguiente equipo autorizado; revisar UDP por separado.
- Evidencia: `versiones-172-17-0-3.nmap`, `versiones-172-17-0-3.gnmap`, `versiones-172-17-0-3.xml`.
![nmap](../capturas/gonzalo/parte1/10.png)
Aqui la comprobación de que se guardaron los tres archivos.
![evidencias](../capturas/gonzalo/parte1/11.png)

## Detección de servicios y versiones — 172.17.0.4
- Cuándo: 17/11/2026, 15:23 (hora local; Nmap registró 14:23 UTC).
- Objetivo y por qué está en alcance: `172.17.0.4`, equipo detectado dentro del rango autorizado `172.17.0.0/24` y confirmado como parte del laboratorio.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T3 -p 22,8009,8080 --reason -oA versiones-172-17-0-4 172.17.0.4`. Consulté solo los puertos TCP previamente encontrados abiertos; no ejecuté scripts NSE ni pruebas de vulnerabilidad.
- Qué salió: `22/tcp` OpenSSH 7.9p1 Debian 10+deb10u4; `8009/tcp` Apache JServ Protocol v1.3, sin versión de producto identificada; `8080/tcp` Apache Tomcat 9.0.30. Nmap reportó Linux.
- Siguiente paso: registrar los productos y versiones observados en `inventario.md` y continuar con el siguiente equipo confirmado; revisar UDP por separado.
- Evidencia: archivos `versiones-172-17-0-4.nmap`, `versiones-172-17-0-4.gnmap`, `versiones-172-17-0-4.xml`.
![nmap](../capturas/gonzalo/parte1/12.png)
Aqui la comprobación de que se guardaron los tres archivos.
![evidencias](../capturas/gonzalo/parte1/13.png)

## Detección de servicios y versiones — 172.17.0.5
- Cuándo: 17/11/2026, 15:28 (hora local; Nmap registró 14:28 UTC).
- Objetivo y por qué está en alcance: `172.17.0.5`, equipo detectado dentro de `172.17.0.0/24` y confirmado como parte del laboratorio autorizado.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T3 -p 80,443,1883,5672,8161,39223,61613,61614,61616 --reason -oA versiones-172-17-0-5 172.17.0.5`. Consulté los puertos TCP abiertos en el escaneo anterior; no ejecuté scripts NSE ni pruebas de vulnerabilidad.
- Qué salió: `80/tcp` y `443/tcp`, Apache httpd 2.4.59 (Debian); `1883/tcp`, MQTT sin versión identificada; `5672/tcp`, posible AMQP, sin identificación confirmada; `8161/tcp` y `61614/tcp`, Jetty 9.4.39.v20210325; `39223/tcp`, tcpwrapped, sin producto identificado; `61613/tcp`, STOMP/Apache ActiveMQ; `61616/tcp`, Apache ActiveMQ OpenWire Transport 5.15.15.
- Siguiente paso: incorporar estos datos y sus incertidumbres a `inventario.md`; continuar con el siguiente equipo y revisar UDP por separado con el ritmo autorizado por el profesor.
- Evidencia: `versiones-172-17-0-5.nmap`, `versiones-172-17-0-5.gnmap`, `versiones-172-17-0-5.xml`.
![nmap](../capturas/gonzalo/parte1/14.png)
Aqui la comprobación de que se guardaron los tres archivos.
![evidencias](../capturas/gonzalo/parte1/15.png)

## Detección de servicios y versiones — 172.17.0.6
- Cuándo: 17/11/2026, 15:32 (hora local; Nmap registró 14:32 UTC).
- Objetivo y por qué está en alcance: `172.17.0.6`, detectado en `172.17.0.0/24` y confirmado como parte de la copia de laboratorio autorizada.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T3 -p 22,8080 --reason -oA versiones-172-17-0-6 172.17.0.6`. Consulté solo los puertos TCP abiertos en el escaneo anterior; no ejecuté scripts NSE ni pruebas de vulnerabilidad.
- Qué salió: `22/tcp` OpenSSH 9.2p1 Debian 2+deb12u2 (protocolo 2.0); `8080/tcp` HTTP, Jetty 10.0.18. Nmap reportó Linux.
- Siguiente paso: registrar estos datos en `inventario.md` y continuar con los siguientes equipos confirmados; revisar UDP por separado.
- Evidencia: `versiones-172-17-0-6.nmap`, `versiones-172-17-0-6.gnmap`, `versiones-172-17-0-6.xml`.
![nmap](../capturas/gonzalo/parte1/16.png)
Aqui la comprobación de que se guardaron los tres archivos.
![evidencias](../capturas/gonzalo/parte1/17.png)

## Detección de servicios y versiones — 172.17.0.7
- Cuándo: 17/11/2026, 15:34 (hora local; Nmap registró 14:34 UTC).
- Objetivo y por qué está en alcance: `172.17.0.7`, detectado en `172.17.0.0/24` y confirmado como parte de la copia de laboratorio autorizada.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T3 -p 22 --reason -oA versiones-172-17-0-7 172.17.0.7`. Consulté solo el puerto TCP que el escaneo anterior encontró abierto; no ejecuté scripts NSE ni pruebas de vulnerabilidad.
- Qué salió: `22/tcp` abierto, servicio SSH, OpenSSH 7.7 (protocolo 2.0). Nmap no indicó distribución o sistema operativo en esta salida.
- Siguiente paso: registrar el resultado en `inventario.md` y continuar con `172.17.0.8`; revisar UDP por separado.
- Evidencia: `versiones-172-17-0-7.nmap`, `versiones-172-17-0-7.gnmap`, `versiones-172-17-0-7.xml`.
![nmap](../capturas/gonzalo/parte1/18.png)
Aqui la comprobación de que se guardaron los tres archivos.
![evidencias](../capturas/gonzalo/parte1/19.png)

## Detección de servicios y versiones — 172.17.0.8
- Cuándo: 17/11/2026, 15:37 (hora local; Nmap registró 14:37 UTC).
- Objetivo y por qué está en alcance: `172.17.0.8`, detectado dentro de `172.17.0.0/24` y confirmado como parte de la copia de laboratorio autorizada.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T3 -p 22,80,139,445 --reason -oA versiones-172-17-0-8 172.17.0.8`. Consulté solo los puertos TCP que el escaneo anterior encontró abiertos; no ejecuté scripts NSE ni pruebas de vulnerabilidad.
- Qué salió: `22/tcp` OpenSSH 9.6p1 Ubuntu 3ubuntu13.5; `80/tcp` Apache httpd 2.4.58 (Ubuntu); `139/tcp` y `445/tcp` Samba smbd 4 (sin versión menor identificada). Nmap reportó Linux.
- Siguiente paso: registrar estos resultados en `inventario.md` y continuar con el siguiente equipo confirmado; revisar UDP por separado.
- Evidencia: `versiones-172-17-0-8.nmap`, `versiones-172-17-0-8.gnmap`, `versiones-172-17-0-8.xml`.
![nmap](../capturas/gonzalo/parte1/20.png)
Aqui la comprobación de que se guardaron los tres archivos.
![evidencias](../capturas/gonzalo/parte1/21.png)

## Detección de servicios y versiones — 172.17.0.9
- Cuándo: 17/11/2026, 15:40 (hora local; Nmap registró 14:40 UTC).
- Objetivo y por qué está en alcance: `172.17.0.9`, detectado en `172.17.0.0/24` y confirmado como parte de la copia de laboratorio autorizada.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T3 -p 22,80 --reason -oA versiones-172-17-0-9 172.17.0.9`. Consulté solo los puertos TCP que el escaneo anterior encontró abiertos; no ejecuté scripts NSE ni pruebas de vulnerabilidad.
- Qué salió: `22/tcp` OpenSSH 9.2p1 Debian 2+deb12u10; `80/tcp` Apache httpd 2.4.68 (Debian). Nmap reportó Linux.
- Siguiente paso: registrar los resultados en `inventario.md` y continuar con `172.17.0.10`; revisar UDP por separado.
- Evidencia: `versiones-172-17-0-9.nmap`, `versiones-172-17-0-9.gnmap`, `versiones-172-17-0-9.xml`.
![nmap](../capturas/gonzalo/parte1/22.png)
Aqui la comprobación de que se guardaron los tres archivos.
![evidencias](../capturas/gonzalo/parte1/23.png)

## Detección de servicios y versiones — 172.17.0.10
- Cuándo: 17/11/2026, 15:51 (hora local; Nmap registró 14:51 UTC).
- Objetivo y por qué está en alcance: `172.17.0.10`, detectado dentro de `172.17.0.0/24` y confirmado como parte de la copia de laboratorio autorizada.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T3 -p 80 --reason -oA versiones-172-17-0-10 172.17.0.10`. Consulté solo el puerto TCP que el escaneo anterior encontró abierto; no ejecuté scripts NSE ni pruebas de vulnerabilidad.
- Qué salió: `80/tcp` abierto, HTTP, SimpleHTTPServer 0.6 (Python 3.11.2).
- Siguiente paso: registrar los servicios y versiones detectados en `inventario.md`; después revisar UDP según la cobertura y el ritmo autorizados por el profesor.
- Evidencia: `versiones-172-17-0-10.nmap`, `versiones-172-17-0-10.gnmap`, `versiones-172-17-0-10.xml`.
![nmap](../capturas/gonzalo/parte1/24.png)
Aqui la comprobación de que se guardaron los tres archivos.
![evidencias](../capturas/gonzalo/parte1/25.png)

## Escaneo de los puertos UDP
- Cuándo: 17/11/2026. Nmap comenzó a las 20:55 UTC es decir las 21:55 de mi equipo; terminó en 1086,47 segundos (unos 18 min 6 s).
- Objetivo y por qué esta en el alcance: los ocho equipos del laboratorio, `172.17.0.3`–`172.17.0.10`, dentro del rango autorizado `172.17.0.0/24`.
- Comando ejecutado: `sudo nmap -sU --top-ports 1000 -T3 --reason -iL hosts_ips_limpio.txt -oA udp`
- Qué salió: revisé los 1.000 puertos UDP más comunes de cada equipo, con Nmap en modo UDP y registrando la razón de cada estado. No ejecuté scripts NSE ni comprobaciones de vulnerabilidades.
- Siguiente paso: comprobar que se guardaron los tres archivos de resultados y enlazarlos como evidencia. Después continuar con la enumeración sin autenticación de los servicios TCP pendientes.
- Evidencia: udp.nmap, udp.gnmap y udp.xml.
![udp.gnmap](../capturas/gonzalo/parte1/27.2.png)
-UDP.NMAP
![udp.nmap](../capturas/gonzalo/parte1/27.3.png)
![udp.nmap](../capturas/gonzalo/parte1/27.4.png)
-UDP.XML
![udp.xml](../capturas/gonzalo/parte1/27.5.png)

## Parte 2 · Enumerar sin autenticar

## Enumeración HTTP sin autenticación — 172.17.0.3
- Cuándo: 17/11/2026, 16:46 (hora local).
- Objetivo y por qué está en alcance: `172.17.0.3:80/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `curl -I --max-time 5 http://172.17.0.3/ | tee enum-172-17-0-3-http.txt` para consultar y guardar las cabeceras HTTP, sin iniciar sesión.
- Qué salió: respuesta `HTTP/1.1 200 OK`; cabecera `Server: Apache/2.4.58 (Ubuntu)`, `Content-Type: text/html` y `Content-Length: 10671`.
- Siguiente paso: revisar qué contenido público ofrece la página y documentar lo que se observa, sin intentar acceder a funciones que pidan autenticación.
- Evidencia: `enum-172-17-0-3-http.txt`.
![nmap](../capturas/gonzalo/parte2/28.png)
![evidencias](../capturas/gonzalo/parte2/29.png)

## Inspección del contenido HTTP — 172.17.0.3
- Cuándo: 17/11/2026, 16:51 (hora local).
- Objetivo y por qué está en alcance: página pública de `172.17.0.3:80/tcp`, dentro de `172.17.0.0/24`.
- Qué hice: inspeccioné localmente el archivo descargado con `less pagina-172-17-0-3.html`; no hice otra petición a la red ni inicié sesión.
- Qué salió: el HTML contiene el título `Apache2 Ubuntu Default Page: It works` y un comentario que indica que la página fue modificada para Ubuntu, con última actualización indicada como 2022-03-22.
- Siguiente paso: anotar que la página predeterminada de Apache está publicada; continuar con el siguiente servicio TCP, sin probar formularios ni credenciales.
- Evidencia: `pagina-172-17-0-3.html`.
![nmap](../capturas/gonzalo/parte2/30.png)
![evidencias](../capturas/gonzalo/parte2/31.png)

## Banner FTP — 172.17.0.3
- Cuándo: 17/11/2026, 17:00 (hora local).
- Objetivo y por qué está en alcance: `172.17.0.3:21/tcp`, servicio FTP detectado dentro de `172.17.0.0/24`.
- Qué hice: ejecuté `nc -nv -w 5 172.17.0.3 21` para leer el saludo del servidor.
- Qué salió: el puerto respondió abierto y el servidor mostró `220 (vsFTPd 2.3.4)`.
- Siguiente paso: continuar con la enumeración sin autenticación de los servicios detectados.
- Evidencia: captura `capturas/28.png`.
![nc](../capturas/gonzalo/parte2/32.png)

## Enumeración FTP sin autenticación — 172.17.0.3
- Cuándo: 19/11/2026, 09:17.
- Objetivo y por qué está en alcance: `172.17.0.3:21/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -sV -T2 -p 21 --script ftp-syst --reason -oA enum-ftp-syst-172-17-0-3 172.17.0.3` para consultar `SYST` y `STAT` sin introducir credenciales.
- Qué salió: Nmap identificó `vsFTPd 2.3.4`; `ftp-syst` no mostró información adicional de `SYST` ni `STAT`.
- Conclusión y limitaciones: `ftp-syst` no aportó información adicional de configuración. No se intentó iniciar sesión, acceder a archivos ni probar credenciales.
- Evidencias: `enum-ftp-syst-172-17-0-3.nmap`, `enum-ftp-syst-172-17-0-3.xml`.
![evidencias](../capturas/gonzalo/parte2/78.png)

## Banner SSH — 172.17.0.4
- Cuándo: 17/11/2026, 17:04 (hora local).
- Objetivo y por qué está en alcance: `172.17.0.4:22/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `nc -nv -w 5 172.17.0.4 22` para leer el banner SSH.
- Qué salió: el servicio presentó `SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u4`. Después apareció `Protocol mismatch`, al no continuar `nc` el intercambio SSH.
- Siguiente paso: consultar las cabeceras HTTP del puerto `8080/tcp` de este equipo, sin autenticación.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:05.
![nc](../capturas/gonzalo/parte2/33.png)

## Enumeración SSH sin autenticación — 172.17.0.4
- Cuándo: 19/11/2026, 09:21.
- Objetivo y por qué está en alcance:`172.17.0.4:22/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -T2 -p 22 --script ssh2-enum-algos,ssh-hostkey --reason -oA enum-ssh-172-17-0-4 172.17.0.4` para consultar los algoritmos SSH ofrecidos y las huellas de las claves públicas, sin iniciar sesión ni probar credenciales.
- Qué salió: el puerto `22/tcp` está abierto. El servidor ofrece varios algoritmos de intercambio de claves, claves de servidor, cifrado, MAC y compresión. La salida incluye `curve25519-sha256`, `ssh-ed25519`, `chacha20-poly1305@openssh.com`, `aes256-gcm@openssh.com`, y compresión `none` y `zlib@openssh.com`. Se detectaron estas huellas:
  - RSA (2048 bits): `4d:8d:56:7f:47:95:da:d9:a4:bb:bc:3e:f1:56:93:d5`
  - ECDSA (256 bits): `8d:82:e6:7d:fb:1c:08:89:06:11:5b:fd:a8:08:1e:72`
  - ED25519 (256 bits): `1e:eb:63:bd:b9:87:72:43:49:6c:76:e1:45:69:ca:75`
- Limitación: la enumeración identifica algoritmos y huellas de claves públicas; no enumera cuentas ni demuestra acceso al sistema.
- Evidencia: `enum-ssh-172-17-0-4.nmap`, `enum-ssh-172-17-0-4.xml`.
![evi](../capturas/gonzalo/parte2/80.png)
![evi](../capturas/gonzalo/parte2/81.png)

## Cabeceras HTTP — 172.17.0.4:8080
- Cuándo: 17/11/2026, 17:06 (hora local aproximada; la respuesta muestra 16:06 GMT).
- Objetivo y por qué está en alcance: `172.17.0.4:8080/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `curl -I --max-time 5 http://172.17.0.4:8080/` para consultar las cabeceras HTTP, sin iniciar sesión.
- Qué salió: respuesta `HTTP/1.1 200`; `Content-Type: text/html; charset=UTF-8`; `Transfer-Encoding: chunked`. Las cabeceras consultadas no muestran el contenido ni identifican la aplicación.
- Siguiente paso: si está permitido, descargar e inspeccionar la página pública; no enviar formularios ni credenciales.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:07.
![nc](../capturas/gonzalo/parte2/34.png)

## Descarga de página HTTP — 172.17.0.4:8080
- Cuándo: 17/11/2026, 17:10 (hora local).
- Objetivo y por qué está en alcance: página principal de `172.17.0.4:8080/tcp`, dentro de `172.17.0.0/24`.
- Qué hice: ejecuté `curl --max-time 5 http://172.17.0.4:8080/ -o pagina-172-17-0-4-8080.html` para guardar localmente la respuesta pública, sin iniciar sesión.
- Qué salió: se recibieron 11.196 bytes y se guardaron en `pagina-172-17-0-4-8080.html`.
- Siguiente paso: inspeccionar el archivo local para identificar el contenido público; no enviar formularios ni credenciales.
- Evidencia: `pagina-172-17-0-4-8080.html` y captura de pantalla del 17/11/2026 a las 17:10.
![nc](../capturas/gonzalo/parte2/35.png)
![evidencia](../capturas/gonzalo/parte2/36.png)

## Identificación de AJP — 172.17.0.4
- Cuándo: 17/11/2026, 23:25 (hora local; Nmap indicó 22:25 UTC).
- Objetivo y por qué está en alcance: `172.17.0.4:8009/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T3 -p 8009 --reason -oA enum-ajp-172-17-0-4` para identificar el servicio, sin autenticación ni scripts NSE.
- Qué salió: `8009/tcp` abierto; servicio `ajp13`, identificado como Apache JServ Protocol v1.3. La comprobación no reveló más información sobre recursos o configuración.
- Siguiente paso: registrar el resultado y su límite en `inventario.md`; continuar con el siguiente servicio pendiente mediante una comprobación específica y prudente.
- Evidencia: `enum-ajp-172-17-0-4.nmap`, `enum-ajp-172-17-0-4.gnmap`, `enum-ajp-172-17-0-4.xml` y la captura `/capturas/58.png`.
![identificación](../capturas/gonzalo/parte2/58.png)
AJP.NMAP
![enum](../capturas/gonzalo/parte2/59.png)
AJP.GNMAP
![enum](../capturas/gonzalo/parte2/60.png)
AJP.XML
![enum](../capturas/gonzalo/parte2/61.png)

## Enumeración AJP sin autenticación — 172.17.0.4
- Cuándo: 19/11/2026, 09:29.
- Objetivo y por qué está en alcance: `172.17.0.4:8009/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -T2 -p 8009 --script ajp-headers --reason -oA enum-ajp-172-17-0-4 172.17.0.4` para solicitar cabeceras HTTP mediante AJP, sin autenticación.
- Qué salió: puerto `8009/tcp` abierto, identificado como `ajp13`. La respuesta mostró `Content-Type: text/html; charset=UTF-8`.
- Limitación: no se obtuvo más información sobre contenido o configuración. No se probaron vulnerabilidades AJP ni se solicitaron archivos.
- Evidencia: `enum-ajp-172-17-0-4.nmap`, `enum-ajp-172-17-0-4.xml`.
![enum](../capturas/gonzalo/parte2/84.png)

## Inspección de la página de Tomcat — 172.17.0.4:8080
- Cuándo: 19/11/2026, 09:31.
- Objetivo y por qué está en alcance: inspección local de la página descargada desde `172.17.0.4:8080/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `grep -Ein '<title>|href=|<h[1-3]' pagina-172-17-0-4-8080.html` para revisar localmente el título, encabezados y enlaces del HTML guardado; no generó nuevas peticiones a la red.
- Qué salió: la página se identifica como `Apache Tomcat/9.0.30` y es la página de bienvenida. Anuncia enlaces a `/docs/`, `/docs/config/`, `/examples/`, `/manager/status`, `/manager/html` y `/host-manager/html`.
- Limitación: el HTML confirma que esas rutas aparecen enlazadas, pero no que sean accesibles ni qué información devuelven. No se abrieron las rutas de administración ni se probaron credenciales.
- Siguiente paso: documentar que esas rutas están enlazadas en la página. No abrir las rutas de administración ni probar credenciales.
- Evidencia: `pagina-172-17-0-4-8080.html`, salida de `grep`.
![enum](../capturas/gonzalo/parte2/85.png)
![enum](../capturas/gonzalo/parte2/86.png)

## Cabeceras HTTP — 172.17.0.5:80
- Cuándo: 17/11/2026, 17:15 (hora local; la respuesta muestra 16:15 GMT).
- Objetivo y por qué está en alcance: `172.17.0.5:80/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `curl -I --max-time 5 http://172.17.0.5/` para consultar las cabeceras HTTP sin iniciar sesión.
- Qué salió: `HTTP/1.1 200 OK`; el servidor declara `Apache/2.4.59 (Debian)`. La respuesta es HTML con `Content-Length: 10701`.
- Siguiente paso: inspeccionar el contenido público de la página principal, sin enviar formularios ni credenciales.
El archivo local contiene el título `Apache2 Debian Default Page: It works`, correspondiente a la página predeterminada de Apache en Debian. El fragmento compartido muestra HTML y estilos; no se observaron cuentas de usuario en ese fragmento.
- Siguiente paso: continuar con otro servicio HTTP identificado, sin enviar formularios ni credenciales.
- Evidencia: `pagina-172-17-0-5.html` y el texto inspeccionado con `less`.
![nc](../capturas/gonzalo/parte2/37.png)
![evidencia](../capturas/gonzalo/parte2/38.png)

## Comprobación HTTPS y certificado — 172.17.0.5:443
- Cuándo: 17/11/2026, 17:22 (hora local); comprobación repetida el 23/11/2026, 10:43 hora local (09:43 GMT).
- Objetivo y por qué está en alcance: `172.17.0.5:443/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté (`curl -k -I --max-time 5 https://172.17.0.5/`) para inspeccionar las cabeceras del servidor, y después descargué la página completa (`curl -k --max-time 5 https://172.17.0.5/`).
- Qué salió: Con `-k` el servidor respondió `HTTP/1.1 200 OK` con `Server: Apache/2.4.59 (Debian)` —corroboración de la versión ya detectada con Nmap—, `Content-Type: text/html` y longitud de contenido 7219; la página es una plantilla de restaurante («DockerLabs», Bootstrap 4.3.x de DevCRUD) guardada como evidencia.
- Siguiente paso: conservar la discrepancia del certificado (`example.com`) como observación de la configuración TLS de `172.17.0.5:443` y continuar con otro servicio que permita una consulta no autenticada.
- Limitación: `-k` omitió únicamente la validación del certificado para inspeccionar contenido público; la discrepancia del nombre del certificado queda documentada como defecto de configuración TLS, no como vulnerabilidad explotada. Las librerías client-side que referencia la página (p. ej. jQuery `3.4.1`) no se valoran como candidatas: el registro cataloga los fallos de los servicios detectados con Nmap.
- Evidencia: [cabeceras con `-k` del 23/11/2026 a las 10:45](../capturas/gonzalo/parte2/39.1.png); página guardada en [`pagina-172-17-0-5-443.html`](../evidencias/gonzalo/pagina-172-17-0-5-443.html).

## Cabeceras HTTP — 172.17.0.5:8161
- Cuándo: 17/11/2026, 17:24 (hora local).
- Objetivo y por qué está en alcance: `172.17.0.5:8161/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `curl -I --max-time 5 http://172.17.0.5:8161/` para consultar las cabeceras sin iniciar sesión.
- Qué salió: respuesta `HTTP/1.1 401 Unauthorized`; el servidor solicita autenticación básica (`realm="ActiveMQReal"`). La cabecera `Server` declara `Jetty(9.4.39.v20210325)`.
- Siguiente paso: registrar que este servicio requiere autenticación y detener la consulta aquí; no probar credenciales. Continuar con otro servicio público.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:24.
![cabeceras](../capturas/gonzalo/parte2/40.png)

## Cabeceras HTTP — 172.17.0.5:61614
- Cuándo: 17/11/2026, 17:25 (hora local; la respuesta muestra 16:25 GMT).
- Objetivo y por qué está en alcance: `172.17.0.5:61614/tcp`, dentro de `172.17.0.0/24`.
- Qué hice: ejecuté `curl -I --max-time 5 http://172.17.0.5:61614/` para consultar las cabeceras, sin iniciar sesión.
- Qué salió: `HTTP/1.1 200 OK`; `Server: Jetty(9.4.39.v20210325)` y `Content-Length: 0`. La respuesta no incluyó contenido.
- Siguiente paso: documentar esta respuesta y continuar con otro servicio TCP; no inferir qué aplicación utiliza este puerto solo por las cabeceras.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:25.
![cabeceras](../capturas/gonzalo/parte2/41.png)

## Enumeración de ActiveMQ y NetBIOS
### ActiveMQ OpenWire — 172.17.0.5
- Cuándo: 17/11/2026, 23:48 (hora local; Nmap indicó 22:47 UTC).
- Objetivo y por qué está en alcance: `172.17.0.5:61616/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T2 -p 61616 --reason -oA enum-openwire-172-17-0-5 172.17.0.5` para identificar el servicio y su versión. No usé scripts NSE ni probé credenciales.
- Qué salió: `61616/tcp` está abierto; Nmap identificó `ActiveMQ OpenWire transport 5.15.15`.
- Siguiente paso: completar `inventario.md` con los resultados de enumeración recogidos y registrar que no se obtuvo información de colas ni mensajes.
- Evidencia: `enum-openwire-172-17-0-5.nmap`, `enum-openwire-172-17-0-5.gnmap`, `enum-openwire-172-17-0-5.xml`.
![nmap](../capturas/gonzalo/parte2/76.png)
NMAP - GNMAP - XML
![nmap-gnmap-xml](../capturas/gonzalo/parte2/77.png)

## Identificación de MQTT — 172.17.0.5
- Cuándo: 17/11/2026, 23:31 (hora local; Nmap indicó 22:31 UTC).
- Objetivo y por qué está en alcance: `172.17.0.5:1883/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T3 -p 1883 --reason -oA enum-mqtt-172-17-0-5 172.17.0.5` para identificar el servicio y su versión. No usé scripts NSE ni probé credenciales.
- Qué salió: `1883/tcp` aparece abierto y Nmap lo identifica como `mqtt`; no identificó una versión.
- Siguiente paso: registrar que el puerto corresponde a MQTT y que la versión quedó sin identificar. Continuar con la identificación ligera de `5672/tcp` en `172.17.0.5`.
- Evidencia: `enum-mqtt-172-17-0-5.nmap`, `enum-mqtt-172-17-0-5.gnmap`, `enum-mqtt-172-17-0-5.xml` y la captura `/capturas/59.png`.
![identificación](../capturas/gonzalo/parte2/62.png)
MQT.NMAP
![namp](../capturas/gonzalo/parte2/63.png)
MQT.GNMAP
![gnmap](../capturas/gonzalo/parte2/64.png)
MQT.XML
![xml](../capturas/gonzalo/parte2/65.png)

## Enumeración MQTT sin autenticación — 172.17.0.5
- Cuándo: 19/11/2026, 09:36.
- Objetivo y por qué está en alcance: `172.17.0.5:1883/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T2 -p 1883 --reason -oA enum-mqtt-172-17-0-5 172.17.0.5` para intentar identificar el servicio y su versión, sin credenciales ni suscripciones.
- Qué salió: el puerto `1883/tcp` está abierto y Nmap lo identifica como `mqtt`; no identificó una versión.
- Siguiente paso: registrar que la versión y la información adicional del broker no se pudieron obtener con esta comprobación. No suscribirse a temas ni leer mensajes.
- Evidencia: `enum-mqtt-172-17-0-5.nmap`, `enum-mqtt-172-17-0-5.xml` y la captura de la salida.
![evi](../capturas/gonzalo/parte2/87.png)

## Identificación y enumeración AMQP — 172.17.0.5
- Cuándo: primera comprobación: 17/11/2026, 22:36 UTC; segunda comprobación: 19/11/2026, 08:43 UTC (09:43, hora local).
- Objetivo y por qué está en alcance: `172.17.0.5:5672/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: primero ejecuté `sudo nmap -sV --version-light -T3 -p 5672 --reason -oA enum-amqp-172-17-0-5 172.17.0.5` para intentar identificar el servicio y su versión. Después ejecuté `sudo nmap -T2 -p 5672 --script amqp-info --reason -oA enum-amqp-info-172-17-0-5 172.17.0.5` para consultar sus propiedades sin credenciales.
- Qué salió: el puerto `5672/tcp` está abierto. La primera prueba lo marcó como `amqp?` y no identificó versión. En la segunda, Nmap lo identificó como `amqp`, pero `amqp-info` informó `AMQP: handshake expected header (1) frame, but was 65`. No se obtuvieron producto, versión ni capacidades; el resultado de la enumeración AMQP es inconcluso.
- Siguiente paso: continuar con la identificación de `172.17.0.5:39223/tcp`, sin enviar más consultas AMQP a este puerto.
- Evidencias: archivos `.nmap`, `.gnmap` y `.xml` de ambas ejecuciones, y capturas correspondientes.
![identificación](../capturas/gonzalo/parte2/66.png)
AMPQ.GNMAP
![gnmap](../capturas/gonzalo/parte2/67.png)
AMPQ.NMAP
![nmap](../capturas/gonzalo/parte2/68.png)
AMPQ.XML
![xml](../capturas/gonzalo/parte2/69.png)
![evi](../capturas/gonzalo/parte2/88.png)

## Identificación de servicio — 172.17.0.5:39223/tcp
- Cuándo: 17/11/2026, 23:42 (hora local; Nmap indicó 22:42 UTC).
- Objetivo y por qué está en alcance: `172.17.0.5:39223/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T2 -p 39223 --reason -oA enum-39223-172-17-0-5 172.17.0.5` para intentar identificar el servicio y su versión. No usé scripts NSE ni probé credenciales.
- Qué salió: el puerto está abierto, pero Nmap lo marcó como `tcpwrapped`; no pudo identificar el servicio ni su versión. Nmap lo identificó como tcpwrapped; no se obtuvo más información.
- Siguiente paso: revisar `172.17.0.5:61613/tcp` con detección ligera de servicio y versión.
- Evidencia: `enum-39223-172-17-0-5.nmap`, `enum-39223-172-17-0-5.gnmap`, `enum-39223-172-17-0-5.xml`.
![nmap](../capturas/gonzalo/parte2/70.png)
NMAP Y GNMAP
![nmap-gnmap](../capturas/gonzalo/parte2/71.png)
XML
![xml](../capturas/gonzalo/parte2/72.png)

## Identificación de STOMP — 172.17.0.5
- Cuándo: 17/11/2026, 23:45 (hora local; Nmap indicó 22:45 UTC).
- Objetivo y por qué está en alcance: `172.17.0.5:61613/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -sV --version-light -T2 -p 61613 --reason -oA enum-stomp-172-17-0-5 172.17.0.5` para identificar el servicio y su versión, sin autenticación ni scripts NSE.
- Qué salió: `61613/tcp` está abierto; Nmap identificó STOMP y Apache ActiveMQ, pero no indicó una versión concreta.
- Siguiente paso: identificar de forma ligera `172.17.0.5:61616/tcp` y documentar si Nmap obtiene versión.
- Evidencia: `enum-stomp-172-17-0-5.nmap`, `enum-stomp-172-17-0-5.gnmap`, `enum-stomp-172-17-0-5.xml`.
![nmap](../capturas/gonzalo/parte2/73.png)
NMAP Y GNMAP
![nmap-gnmap](../capturas/gonzalo/parte2/74.png)
XML
![xml](../capturas/gonzalo/parte2/75.png)

## Comprobación de conexión STOMP anónima — 172.17.0.5
- Cuándo: 19/11/2026, 09:56 (hora local).
- Objetivo y por qué está en alcance: `172.17.0.5:61613/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: envié un saludo `CONNECT` de STOMP sin campos de usuario ni contraseña con `printf 'CONNECT\naccept-version:1.2\nhost:localhost\n\n\0' | nc -nv -w 5 172.17.0.5 61613 2>&1 | tee enum-stomp-anon-172-17-0-5.txt`.
- Qué salió: el servidor respondió `CONNECTED`, anunció `ActiveMQ/5.15.15` y aceptó STOMP `1.2`. Esto confirma que aceptó la conexión sin credenciales.
- Limitación y alcance: no se enviaron comandos `SUBSCRIBE` ni `SEND`; por tanto, no se comprobó acceso a colas ni se leyeron o publicaron mensajes. Se detuvo la prueba tras recibir la respuesta.
- Siguiente paso: registrar este hallazgo y comunicar al profesor que STOMP aceptó una conexión anónima; no continuar interactuando con el broker.
- Evidencia: `enum-stomp-anon-172-17-0-5.txt`.
![compro](../capturas/gonzalo/parte2/89.png)

## Banner SSH — 172.17.0.6
- Cuándo: 17/11/2026, 17:27 (hora local).
- Objetivo y por qué está en alcance: `172.17.0.6:22/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `nc -nv -w 5 172.17.0.6 22` para leer el banner SSH.
- Qué salió: el servicio presentó `SSH-2.0-OpenSSH_9.2p1 Debian-2+deb12u2`.
- Siguiente paso: registrar el banner y continuar con otro servicio TCP sin autenticación.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:27.
![banner](../capturas/gonzalo/parte2/42.png)

## Enumeración SSH sin autenticación — 172.17.0.6
- Cuándo: 19/11/2026, 08:25 UTC.
- Objetivo y por qué está en alcance: `172.17.0.6:22/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -T2 -p 22 --script ssh2-enum-algos,ssh-hostkey --reason -oA enum-ssh-172-17-0-6 172.17.0.6` para identificar los algoritmos SSH ofrecidos y las huellas de las claves públicas, sin iniciar sesión ni probar credenciales.
- Qué salió: el puerto `22/tcp` está abierto. El servidor ofrece algoritmos de intercambio de claves, claves de servidor, cifrado, MAC y compresión. Entre los identificados aparecen `curve25519-sha256`, `ssh-ed25519`, `chacha20-poly1305@openssh.com`, `aes256-gcm@openssh.com` y compresión `none` y `zlib@openssh.com`. Huellas de claves públicas:
  - ECDSA (256 bits): `94:fb:28:59:7f:ae:02:c0:56:46:07:33:8c:ac:52:85`
  - ED25519 (256 bits): `43:07:50:30:bb:28:b0:73:9b:7c:0c:4e:3f:c9:bf:02`
- Limitación: la comprobación revela algoritmos y huellas públicas, pero no enumera cuentas ni demuestra acceso al sistema.
- Evidencia: `enum-ssh-172-17-0-6.nmap`, `enum-ssh-172-17-0-6.xml`.

## Cabeceras HTTP — 172.17.0.6:8080
- Cuándo: 17/11/2026, 17:29 (hora local; la respuesta indica 16:29 GMT).
- Objetivo y por qué está en alcance: `172.17.0.6:8080/tcp`, dentro de `172.17.0.0/24`.
- Qué hice: ejecuté `curl -I --max-time 5 http://172.17.0.6:8080/` para consultar las cabeceras, sin iniciar sesión.
- Qué salió: respuesta `HTTP/1.1 403 Forbidden`. Las cabeceras muestran `Server: Jetty(10.0.18)`, `X-Jenkins: 2.441` y `X-Hudson: 1.395`. También se envió una cookie de sesión; su valor se omite del registro.
- Siguiente paso: documentar la información visible y la limitación de acceso; no intentar sortear el 403. Continuar con otro servicio que permita consulta pública.
- Evidencia: captura del 17/11/2026 a las 17:29.
![cabecera](../capturas/gonzalo/parte2/43.png)

## Banner SSH — 172.17.0.7
- Cuándo: 17/11/2026, 17:32 (hora local).
- Objetivo y por qué está en alcance: `172.17.0.7:22/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `nc -nv -w 5 172.17.0.7 22` para leer el banner SSH.
- Qué salió: el servidor presentó `SSH-2.0-OpenSSH_7.7`. Después apareció `Protocol mismatch` porque `nc` no continúa el intercambio SSH.
- Siguiente paso: continuar con `172.17.0.8`, consultando solo el listado SMB sin contraseña.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:32.
![banner](../capturas/gonzalo/parte2/44.png)

## Enumeración SSH sin autenticación — 172.17.0.7
- Cuándo: 19/11/2026, 10:06 (hora local; Nmap indicó 09:06 UTC).
- Objetivo y por qué está en alcance: `172.17.0.7:22/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -T2 -p 22 --script ssh2-enum-algos,ssh-hostkey --reason -oA enum-ssh-172-17-0-7 172.17.0.7` para enumerar algoritmos y huellas de las claves públicas, sin iniciar sesión ni probar credenciales.
- Qué salió: el puerto `22/tcp` está abierto. Se identificaron algoritmos de intercambio de claves, cifrado, MAC y compresión. Huellas de claves públicas:
  - RSA (2048 bits): `1a:cb:5e:a3:3d:d1:da:c0:ed:2a:61:7f:73:79:46:ce`
  - ECDSA (256 bits): `54:9e:53:23:57:fc:60:1e:c0:41:cb:f3:85:32:01:fc`
  - ED25519 (256 bits): `4b:15:7e:7b:b3:07:54:3d:74:ad:e0:94:78:0c:94:93`
- Limitación: no se enumeraron cuentas ni se probó el acceso al sistema.
- Siguiente paso: repetir esta enumeración en `172.17.0.8:22`.
- Evidencia: `enum-ssh-172-17-0-7.nmap`, `enum-ssh-172-17-0-7.xml`.
![cabecera](../capturas/gonzalo/parte2/91.png)
![cabecera](../capturas/gonzalo/parte2/92.png)

## Listado SMB sin contraseña — 172.17.0.8
- Cuándo: 17/11/2026, 17:34 (hora local).
- Objetivo y por qué está en alcance: `172.17.0.8:445/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `smbclient -N -L //172.17.0.8` para pedir el listado SMB sin contraseña; no accedí a los recursos compartidos.
- Qué salió: sesión anónima aceptada. Se listaron `myshare` (“Carpeta compartida sin restricciones”), `backup24` (“Privado”), `home` (“Produccion”) e `IPC$` (“IPC Service (EseEmeB Samba Server)”). Después falló la negociación SMB1 para el listado del grupo de trabajo (`NT_STATUS_INVALID_NETWORK_RESPONSE`). Los comentarios no prueban los permisos efectivos de los recursos.
- Siguiente paso: documentar el listado y la limitación; no abrir los recursos. Consultar si SMB permite revelar cuentas mediante una consulta nula autorizada.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:35.
![cabecera](../capturas/gonzalo/parte2/45.png)

## Enumeración de cuentas SMB sin credenciales — 172.17.0.8
- Cuándo: 17/11/2026, 17:38 (hora local).
- Objetivo y por qué está en alcance: `172.17.0.8:445/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `rpcclient -U "" -N 172.17.0.8 -c enumdomusers` para solicitar cuentas por RPC sin usuario ni contraseña.
- Qué salió: el servicio reveló `usuario1` (RID `0x3e8`), `usuario3` (`0x3ea`), `administrador` (`0x3ec`), `usuario2` (`0x3e9`) y `Satriani7` (`0x3eb`). Esto solo confirma que SMB expuso esos nombres; no que se pueda autenticar con ellos.
- Siguiente paso: registrar las cuentas y la evidencia; continuar con la web pública de `172.17.0.8:80` sin autenticación.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:38.
![enumeracion](../capturas/gonzalo/parte2/46.png)

## Cabeceras HTTP — 172.17.0.8:80
- Cuándo: 17/11/2026, 17:39 (hora local; la respuesta muestra 16:39 GMT).
- Objetivo y por qué está en alcance: `172.17.0.8:80/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `curl -I --max-time 5 http://172.17.0.8/` para consultar las cabeceras HTTP sin iniciar sesión.
- Qué salió: respuesta `HTTP/1.1 200 OK`; el servidor declara `Apache/2.4.58 (Ubuntu)` y `Content-Type: text/html; charset=UTF-8`.
- Siguiente paso: inspeccionar el contenido público de la página principal, sin enviar formularios ni credenciales.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:39.
![cabeceras](../capturas/gonzalo/parte2/47.png)

## Descarga de página HTTP — 172.17.0.8:80
- Cuándo: 17/11/2026, aproximadamente 17:40.
- Objetivo y por qué está en alcance: página principal de `172.17.0.8:80/tcp`, dentro de `172.17.0.0/24`.
- Qué hice: ejecuté `curl --max-time 5 http://172.17.0.8/ -o pagina-172-17-0-8.html` para guardar la respuesta pública localmente; no envié formularios ni credenciales.
- Qué salió: se guardó la página HTML en `pagina-172-17-0-8.html`.
- Siguiente paso: inspeccionar el archivo localmente.
- Evidencia: `pagina-172-17-0-8.html` y la salida de terminal compartida.
![cabeceras](../capturas/gonzalo/parte2/48.png)

## Inspección local de página HTTP — 172.17.0.8
- Cuándo: 17/11/2026, aproximadamente 17:40.
- Objetivo y por qué está en alcance: inspección local de la respuesta previamente obtenida de `172.17.0.8:80/tcp`.
- Qué hice: abrí `pagina-172-17-0-8.html` con `less`; no hice otra petición a la red.
- Qué salió: el título es `Login`. El HTML muestra campos de correo y contraseña, un formulario `POST` y un enlace “Regístrate”. No se enviaron datos ni se observaron nombres de cuentas en el fragmento compartido.
- Siguiente paso: documentar que la página presenta un formulario y detener la interacción con él; continuar con otro servicio público.
- Evidencia: `pagina-172-17-0-8.html` y el texto inspeccionado.
![inspeccion](../capturas/gonzalo/parte2/49.png)

### NetBIOS — 172.17.0.8
- Cuándo: 19/11/2026, 10:04.
- Objetivo y por qué está en alcance: `172.17.0.8:137/udp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `nmblookup -A 172.17.0.8 | tee enum-netbios-172-17-0-8.txt` para consultar los nombres NetBIOS publicados por el equipo, sin iniciar sesión.
- Qué salió: se anunciaron los nombres `SAMBASERVER` y `ESEEMEB.DL`, además de entradas asociadas a los servicios de servidor y explorador. La consulta no enumeró archivos ni cuentas.
- Siguiente paso: conservar el resultado como información de identificación NetBIOS. El puerto `138/udp` quedó `open|filtered` y no respondió en el escaneo; no se obtuvo información adicional.
- Evidencia: `enum-netbios-172-17-0-8.txt`.
![evi](../capturas/gonzalo/parte2/90.png)

## Enumeración SSH sin autenticación — 172.17.0.8
- Cuándo: 19/11/2026, 10:15 (hora local; Nmap indicó 09:15 UTC).
- Objetivo y por qué está en alcance:** `172.17.0.8:22/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -T2 -p 22 --script ssh2-enum-algos,ssh-hostkey --reason -oA enum-ssh-172-17-0-8 172.17.0.8` para consultar algoritmos SSH y huellas de claves públicas, sin iniciar sesión ni probar credenciales.
- Qué salió: el puerto `22/tcp` está abierto. Se enumeraron algoritmos de intercambio de claves, claves de servidor, cifrado, MAC y compresión. Huellas de claves públicas:
  - ECDSA (256 bits): `43:a1:09:2d:be:05:58:1b:01:20:d7:d0:d8:0d:7b:a6`
  - ED25519 (256 bits): `cd:98:0b:8a:0b:f9:f5:43:e4:44:5d:33:2f:08:2e:ce`
- Limitación: no se enumeraron cuentas ni se probó el acceso al sistema.
- Siguiente paso: repetir esta enumeración en `172.17.0.9:22`.
- Evidencia: `enum-ssh-172-17-0-8.nmap`, `enum-ssh-172-17-0-8.xml`.
![evi](../capturas/gonzalo/parte2/95.png)
![evi](../capturas/gonzalo/parte2/96.png)

## Banner SSH — 172.17.0.9
- Cuándo: 17/11/2026, 17:43 (hora local).
- Objetivo y por qué está en alcance: `172.17.0.9:22/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `nc -nv -w 5 172.17.0.9 22` para leer el banner SSH.
- Qué salió: el servidor presentó `SSH-2.0-OpenSSH_9.2p1 Debian-2+deb12u10`. Después apareció `Invalid SSH identification string` porque `nc` no continúa el protocolo SSH.
- Siguiente paso: consultar las cabeceras HTTP públicas de `172.17.0.9:80`.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:43.
![banner](../capturas/gonzalo/parte2/50.png)

## Enumeración SSH sin autenticación — 172.17.0.9
- Cuándo: 19/11/2026, 10:20 (hora local; Nmap indicó 09:20 UTC).
- Objetivo y por qué está en alcance: `172.17.0.9:22/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -T2 -p 22 --script ssh2-enum-algos,ssh-hostkey --reason -oA enum-ssh-172-17-0-9 172.17.0.9` para consultar los algoritmos ofrecidos y las huellas de las claves públicas, sin iniciar sesión ni probar credenciales.
- Qué salió: el puerto `22/tcp` está abierto. Se identificaron algoritmos de intercambio de claves, claves de servidor, cifrado, MAC y compresión. Huellas de claves públicas:
  - ECDSA (256 bits): `64:3e:fd:2e:0a:65:fe:93:af:79:39:4f:bb:95:26:fa`
  - ED25519 (256 bits): `c2:21:1e:90:e8:06:5d:b0:94:07:f0:07:26:36:a7:23`
- Limitación: esta consulta no enumera cuentas ni prueba el acceso al sistema.
- Siguiente paso: registrar que SSH expuso algoritmos y huellas públicas; no intentar inicios de sesión.
- Evidencia: `enum-ssh-172-17-0-9.nmap`, `enum-ssh-172-17-0-9.xml`.

## Cabeceras HTTP — 172.17.0.9:80
- Cuándo: 17/11/2026, 17:44 (hora local; la respuesta muestra 16:44 GMT).
- Objetivo y por qué está en alcance: `172.17.0.9:80/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `curl -I --max-time 5 http://172.17.0.9/` para consultar las cabeceras, sin iniciar sesión.
- Qué salió: `HTTP/1.1 200 OK`; el servidor declara `Apache/2.4.68 (Debian)`, con contenido HTML y longitud de 537 bytes.
- Siguiente paso: guardar e inspeccionar localmente la página principal, sin enviar formularios ni credenciales.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:45.
![cabecera](../capturas/gonzalo/parte2/51.png)

## Descarga de página HTTP — 172.17.0.9:80
- Cuándo: 17/11/2026, 17:46 (hora local).
- Objetivo y por qué está en alcance: página principal de `172.17.0.9:80/tcp`, dentro de `172.17.0.0/24`.
- Qué hice: ejecuté `curl --max-time 5 http://172.17.0.9/ -o pagina-172-17-0-9.html` para guardar la respuesta pública localmente, sin enviar formularios ni credenciales.
- Qué salió: se recibieron 537 bytes y se guardaron en `pagina-172-17-0-9.html`.
- Siguiente paso: inspeccionar el archivo local para describir el contenido público.
- Evidencia: `pagina-172-17-0-9.html` y captura de pantalla del 17/11/2026 a las 17:46.
![cabecera](../capturas/gonzalo/parte2/52.png)

## Inspección local de página HTTP — 172.17.0.9
- Cuándo: 17/11/2026, 17:46.
- Objetivo y por qué está en alcance: inspección local de la página obtenida de `172.17.0.9:80/tcp`, dentro de `172.17.0.0/24`.
- Qué hice: abrí `pagina-172-17-0-9.html` con `less`; no hice otra petición a la red.
- Qué salió: título `test page`, mensaje `Under Development` y enlaces `Home Page` y `Login`. No se observaron cuentas en el fragmento compartido.
- Siguiente paso: registrar lo observado; no abrir el enlace de Login ni enviar credenciales. Continuar con el siguiente equipo.
- Evidencia: `pagina-172-17-0-9.html`.
![cabecera](../capturas/gonzalo/parte2/53.png)

## Cabeceras HTTP — 172.17.0.10:80
- Cuándo: 17/11/2026, 17:48 (hora local; la respuesta indica 16:48 GMT).
- Objetivo y por qué está en alcance: `172.17.0.10:80/tcp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `curl -I --max-time 5 http://172.17.0.10/` para consultar las cabeceras, sin iniciar sesión.
- Qué salió: `HTTP/1.0 200 OK`; el servidor declara `SimpleHTTP/0.6 Python/3.11.2`; `Content-Type: text/html`; longitud 10.398 bytes.
- Siguiente paso: guardar y revisar localmente el contenido público de la página principal; no enviar formularios ni credenciales.
- Evidencia: captura de pantalla del 17/11/2026 a las 17:48.
![cabecera](../capturas/gonzalo/parte2/54.png)

## Descarga de página HTTP — 172.17.0.10:80
- Cuándo: 17/11/2026, 17:49 (hora local, según la captura).
- Objetivo y por qué está en alcance: página principal de `172.17.0.10:80/tcp`, dentro de `172.17.0.0/24`.
- Qué hice: ejecuté `curl --max-time 5 http://172.17.0.10/ -o pagina-172-17-0-10.html` para guardar localmente la respuesta pública, sin enviar formularios ni credenciales.
- Qué salió: se recibieron 10.398 bytes y se guardaron en `pagina-172-17-0-10.html`.
- Siguiente paso: inspeccionar el archivo local para describir el contenido público.
- Evidencia: `pagina-172-17-0-10.html`.
![cabecera](../capturas/gonzalo/parte2/55.png)

## Inspección local de página HTTP — 172.17.0.10
- Cuándo: 17/11/2026, 17:51.
- Objetivo y por qué está en alcance: inspección local de `pagina-172-17-0-10.html`, descargada desde `172.17.0.10:80/tcp`, dentro de `172.17.0.0/24`.
- Qué hice: inspeccioné con `less` el archivo guardado localmente; no realicé otra petición a la red.
- Qué salió: el fragmento muestra el título `CTG · IPMI bajo exposición` y una página en español. El contenido principal no está incluido en el fragmento compartido, por lo que queda pendiente de observar.
- Siguiente paso: revisar localmente el resto del HTML y anotar solo el texto y enlaces visibles; no interactuar con formularios o funciones de gestión.
- Evidencia: `pagina-172-17-0-10.html`.
![inspeccion](../capturas/gonzalo/parte2/56.png)

## Inspección local de página HTTP — 172.17.0.10
- Cuándo: 17/11/2026; hora exacta no visible en la salida.
- Objetivo y por qué está en alcance: inspección local del HTML obtenido de `172.17.0.10:80/tcp`, dentro de `172.17.0.0/24`.
- Qué hice: ejecuté `grep -n -A 80 '\<body' pagina-172-17-0-10.html` para leer localmente el contenido HTML; no hice otra petición a la red.
- Qué salió: la página titulada `CTG · IPMI bajo exposición` explica IPMI, BMC, SOL y menciona UDP 623 como puerto habitual. Es contenido informativo y no confirma que IPMI esté activo en este equipo.
- Siguiente paso: documentar el contenido como información publicada por la web; no atribuir IPMI ni UDP 623 al host sin evidencia de red.
- Evidencia: `pagina-172-17-0-10.html` y la salida de `grep`.
![inspeccion](../capturas/gonzalo/parte2/57.png)

## Enumeración IPMI sin autenticación — 172.17.0.10
- Cuándo: 19/11/2026, 10:25 (hora local; Nmap indicó 09:25 UTC).
- Objetivo y por qué está en alcance: `172.17.0.10:623/udp`, dentro del rango autorizado `172.17.0.0/24`.
- Qué hice: ejecuté `sudo nmap -sU -T2 -p 623 --script ipmi-version --reason -oA enum-ipmi-172-17-0-10 172.17.0.10` para identificar la versión IPMI y los métodos de autenticación anunciados, sin probar credenciales.
- Qué salió: `623/udp` está abierto. Nmap identificó IPMI `2.0` y reportó los métodos `password`, `md5`, `md2` y `null` en `UserAuth`; también mostró `auth_msg`, `auth_user` y `non_null_user` en `PassAuth`.
- Limitación y precaución: que aparezca `null` no confirma que se pueda administrar el BMC sin autenticación. No se intentó iniciar sesión ni acceder a funciones de gestión. Se detienen las pruebas de IPMI y se informa al profesor para que valore el indicio.
- Evidencia: `enum-ipmi-172-17-0-10.nmap`, `enum-ipmi-172-17-0-10.xml`.
![enumer](../capturas/gonzalo/parte2/97.png)

### Archivos de evidencia guardados en `evidencias/`

Salidas asociadas a las comprobaciones de esta parte:

- **FTP — 172.17.0.3:21:** [enum-ftp-syst-172-17-0-3.nmap](../evidencias/gonzalo/enum-ftp-syst-172-17-0-3.nmap), [enum-ftp-syst-172-17-0-3.gnmap](../evidencias/gonzalo/enum-ftp-syst-172-17-0-3.gnmap), [enum-ftp-syst-172-17-0-3.xml](../evidencias/gonzalo/enum-ftp-syst-172-17-0-3.xml).
- **HTTP — 172.17.0.3:80:** [enum-172-17-0-3-http.txt](../evidencias/gonzalo/enum-172-17-0-3-http.txt), [pagina-172-17-0-3.html](../evidencias/gonzalo/pagina-172-17-0-3.html).
- **SSH — 172.17.0.4:22:** [enum-ssh-172-17-0-4.nmap](../evidencias/gonzalo/enum-ssh-172-17-0-4.nmap), [enum-ssh-172-17-0-4.gnmap](../evidencias/gonzalo/enum-ssh-172-17-0-4.gnmap), [enum-ssh-172-17-0-4.xml](../evidencias/gonzalo/enum-ssh-172-17-0-4.xml).
- **AJP — 172.17.0.4:8009:** [enum-ajp-172-17-0-4.nmap](../evidencias/gonzalo/enum-ajp-172-17-0-4.nmap), [enum-ajp-172-17-0-4.gnmap](../evidencias/gonzalo/enum-ajp-172-17-0-4.gnmap), [enum-ajp-172-17-0-4.xml](../evidencias/gonzalo/enum-ajp-172-17-0-4.xml).
- **HTTP/Tomcat — 172.17.0.4:8080:** [pagina-172-17-0-4-8080.html](../evidencias/gonzalo/pagina-172-17-0-4-8080.html).
- **HTTP — 172.17.0.5:80:** [pagina-172-17-0-5.html](../evidencias/gonzalo/pagina-172-17-0-5.html).
- **HTTPS — 172.17.0.5:443:** [pagina-172-17-0-5-443.html](../evidencias/gonzalo/pagina-172-17-0-5-443.html).
- **MQTT — 172.17.0.5:1883:** [enum-mqtt-172-17-0-5.nmap](../evidencias/gonzalo/enum-mqtt-172-17-0-5.nmap), [enum-mqtt-172-17-0-5.gnmap](../evidencias/gonzalo/enum-mqtt-172-17-0-5.gnmap), [enum-mqtt-172-17-0-5.xml](../evidencias/gonzalo/enum-mqtt-172-17-0-5.xml).
- **AMQP — 172.17.0.5:5672:** [enum-amqp-172-17-0-5.nmap](../evidencias/gonzalo/enum-amqp-172-17-0-5.nmap), [enum-amqp-172-17-0-5.gnmap](../evidencias/gonzalo/enum-amqp-172-17-0-5.gnmap), [enum-amqp-172-17-0-5.xml](../evidencias/gonzalo/enum-amqp-172-17-0-5.xml), [enum-amqp-info-172-17-0-5.nmap](../evidencias/gonzalo/enum-amqp-info-172-17-0-5.nmap), [enum-amqp-info-172-17-0-5.gnmap](../evidencias/gonzalo/enum-amqp-info-172-17-0-5.gnmap), [enum-amqp-info-172-17-0-5.xml](../evidencias/gonzalo/enum-amqp-info-172-17-0-5.xml).
- **Servicio no identificado — 172.17.0.5:39223:** [enum-39223-172-17-0-5.nmap](../evidencias/gonzalo/enum-39223-172-17-0-5.nmap), [enum-39223-172-17-0-5.gnmap](../evidencias/gonzalo/enum-39223-172-17-0-5.gnmap), [enum-39223-172-17-0-5.xml](../evidencias/gonzalo/enum-39223-172-17-0-5.xml).
- **STOMP — 172.17.0.5:61613:** [enum-stomp-172-17-0-5.nmap](../evidencias/gonzalo/enum-stomp-172-17-0-5.nmap), [enum-stomp-172-17-0-5.gnmap](../evidencias/gonzalo/enum-stomp-172-17-0-5.gnmap), [enum-stomp-172-17-0-5.xml](../evidencias/gonzalo/enum-stomp-172-17-0-5.xml), [enum-stomp-anon-172-17-0-5.txt](../evidencias/gonzalo/enum-stomp-anon-172-17-0-5.txt).
- **OpenWire — 172.17.0.5:61616:** [enum-openwire-172-17-0-5.nmap](../evidencias/gonzalo/enum-openwire-172-17-0-5.nmap), [enum-openwire-172-17-0-5.gnmap](../evidencias/gonzalo/enum-openwire-172-17-0-5.gnmap), [enum-openwire-172-17-0-5.xml](../evidencias/gonzalo/enum-openwire-172-17-0-5.xml).
- **SSH — 172.17.0.6:22:** [enum-ssh-172-17-0-6.nmap](../evidencias/gonzalo/enum-ssh-172-17-0-6.nmap), [enum-ssh-172-17-0-6.gnmap](../evidencias/gonzalo/enum-ssh-172-17-0-6.gnmap), [enum-ssh-172-17-0-6.xml](../evidencias/gonzalo/enum-ssh-172-17-0-6.xml).
- **SSH — 172.17.0.7:22:** [enum-ssh-172-17-0-7.nmap](../evidencias/gonzalo/enum-ssh-172-17-0-7.nmap), [enum-ssh-172-17-0-7.gnmap](../evidencias/gonzalo/enum-ssh-172-17-0-7.gnmap), [enum-ssh-172-17-0-7.xml](../evidencias/gonzalo/enum-ssh-172-17-0-7.xml).
- **SSH — 172.17.0.8:22:** [enum-ssh-172-17-0-8.nmap](../evidencias/gonzalo/enum-ssh-172-17-0-8.nmap), [enum-ssh-172-17-0-8.gnmap](../evidencias/gonzalo/enum-ssh-172-17-0-8.gnmap), [enum-ssh-172-17-0-8.xml](../evidencias/gonzalo/enum-ssh-172-17-0-8.xml).
- **HTTP — 172.17.0.8:80:** [pagina-172-17-0-8.html](../evidencias/gonzalo/pagina-172-17-0-8.html).
- **NetBIOS — 172.17.0.8:137/udp:** [enum-netbios-172-17-0-8.txt](../evidencias/gonzalo/enum-netbios-172-17-0-8.txt).
- **SSH — 172.17.0.9:22:** [enum-ssh-172-17-0-9.nmap](../evidencias/gonzalo/enum-ssh-172-17-0-9.nmap), [enum-ssh-172-17-0-9.gnmap](../evidencias/gonzalo/enum-ssh-172-17-0-9.gnmap), [enum-ssh-172-17-0-9.xml](../evidencias/gonzalo/enum-ssh-172-17-0-9.xml).
- **HTTP — 172.17.0.9:80:** [pagina-172-17-0-9.html](../evidencias/gonzalo/pagina-172-17-0-9.html).
- **HTTP — 172.17.0.10:80:** [pagina-172-17-0-10.html](../evidencias/gonzalo/pagina-172-17-0-10.html).
- **UDP/IPMI — 172.17.0.10:623:** [enum-ipmi-172-17-0-10.nmap](../evidencias/gonzalo/enum-ipmi-172-17-0-10.nmap), [enum-ipmi-172-17-0-10.gnmap](../evidencias/gonzalo/enum-ipmi-172-17-0-10.gnmap), [enum-ipmi-172-17-0-10.xml](../evidencias/gonzalo/enum-ipmi-172-17-0-10.xml).
- **Escaneo UDP / comprobaciones NetBIOS:** [udp.nmap](../evidencias/gonzalo/udp.nmap), [udp.gnmap](../evidencias/gonzalo/udp.gnmap), [udp.xml](../evidencias/gonzalo/udp.xml).
- **SMB anónimo y enumeración de cuentas — `172.17.0.8:445`:** la evidencia conservada son las capturas [listado SMB](../capturas/gonzalo/parte2/45.png) y [cuentas](../capturas/gonzalo/parte2/46.png); no hay una salida de texto separada en `evidencias/`.

## Parte 3 · De la versión al CVE

- **Cuándo:** 20/11/2026 10:43 hora local hasta 12:18.
- **Objetivo y por qué está en alcance:** analizar posibles vulnerabilidades de las versiones observadas en `172.17.0.3`–`172.17.0.10`, dentro del rango autorizado `172.17.0.0/24`.
- **Qué hice:** revisé los campos `<service>` y `<cpe>` de los XML de Nmap y contrasté producto/versión con NVD y avisos de fabricantes o distribuciones. Para cada candidato anoté CVE, explicación, CWE, técnica ATT&CK, evidencia, condiciones de aplicación y limitaciones. Consulté CVSS, EPSS y CISA KEV como datos del registro, sin asignar prioridad en esta fase. No probé vulnerabilidades ni envié comprobaciones a los servicios para confirmar fallos.
- **Qué salió:** preparé doce fichas de candidatos en `registro-vulns.md`. Las fichas se consideran posibilidades, no vulnerabilidades confirmadas; la versión o el CPE por sí solos no prueban que el paquete instalado carezca de parches ni que se cumplan las condiciones de configuración. Dejé aparte los falsos positivos o casos sin versión suficiente, explicando el motivo.
- **Siguiente paso:** conservar `registro-vulns.md` como borrador del registro de Parte 3. Si en una fase siguiente se autoriza nuevas comprobaciones, verificar únicamente las revisiones de paquete y condiciones de configuración permitidas.
- **Evidencias:** [registro de vulnerabilidades](../registro-vulns.md); XML de versiones de Nmap: [`.3`](../evidencias/gonzalo/versiones-172-17-0-3.xml), [`.4`](../evidencias/gonzalo/versiones-172-17-0-4.xml), [`.5`](../evidencias/gonzalo/versiones-172-17-0-5.xml), [`.6`](../evidencias/gonzalo/versiones-172-17-0-6.xml), [`.7`](../evidencias/gonzalo/versiones-172-17-0-7.xml), [`.8`](../evidencias/gonzalo/versiones-172-17-0-8.xml), [`.9`](../evidencias/gonzalo/versiones-172-17-0-9.xml) y [`.10`](../evidencias/gonzalo/versiones-172-17-0-10.xml); resultados de EPSS/KEV: [todos los CVE consultados](../evidencias/gonzalo/epss-kev-todos.txt) y [V-11/V-12](../evidencias/gonzalo/epss-kev-ssh-jetty.txt). Capturas de apoyo: [resumen de versiones](../capturas/gonzalo/parte3/1.png)y [consulta de EPSS/KEV](../capturas/gonzalo/parte3/8.png)[consulta de EPSS/KEV](../capturas/gonzalo/parte3/8.1.1.png).

## Parte 4 · Priorizar e informar

- **Cuándo:** 20/11/2026 12:23 a 15:35.
- **Objetivo y por qué está en alcance:** prioriza los hallazgos de los servicios observados en `172.17.0.3`–`172.17.0.10` y responde a la consulta del cliente sobre CVE-2026-123158, dentro del alcance autorizado `172.17.0.0/24`.
- **Qué hice:** contrasté los CVSS y sus vectores C/I/A con NVD/CNA, revisé EPSS de FIRST (fecha de los datos: 19/11/2026) y el catálogo CISA KEV versión `2026.11.18`, incluidos sus campos de uso en ransomware. Ordené los candidatos considerando explotación conocida, impacto, puntuación, EPSS, servicio/versión observados y condiciones todavía no verificadas. Revisé el aviso «SSHole» con la ficha NVD, las notas oficiales de OpenSSH y el aviso de Alpine.
- **Qué salió:** añadí a [registro-vulns.md](../registro-vulns.md) la priorización de Parte 4 y la ficha V-13 para SSHole. La CVE está en KEV desde el 09/11/2026, tiene EPSS 18,9 % (dato del 19/11) y el campo de ransomware aparece como `Unknown`. Los banners muestran cinco servicios SSH anteriores a `10.6p2`, pero no revelan libc, PAM ni la configuración/cuentas necesarias para determinar aplicabilidad; por tanto, no concluyo que los equipos sean vulnerables. Preparé [informe-cliente.md](../informe-cliente.md) con el orden recomendado y la respuesta al cliente.
- **Siguiente paso:** que el cliente/administrador autorizado verifique en los hosts las revisiones de paquetes y condiciones de configuración señaladas; no hacer pruebas de acceso ni explotación dentro de esta fase.
- **Evidencias:** [registro de vulnerabilidades](../registro-vulns.md), [informe para el cliente](../informe-cliente.md), [EPSS y consultas KEV](../evidencias/gonzalo/epss-kev-todos.txt), y los XML/Nmap de versiones enlazados en V-13. Fuentes: [NVD CVE-2026-123158](https://nvd.nist.gov/vuln/detail/CVE-2026-123158), [OpenSSH 10.6p2](https://www.openssh.org/releasenotes.html#10.6p2), [Alpine Linux](https://alpinelinux.org/posts/2026-11-05-openssh-password-authentication-bypass/) y [catálogo CISA KEV](https://github.com/cisagov/kev-data/blob/develop/known_exploited_vulnerabilities.json).

## Valoración de CVE-2013-4786 (IPMI 2.0) — 172.17.0.10

- **Cuándo:** 23/11/2026, 11:40 (hora local).
- **Objetivo y por qué está en alcance:** `172.17.0.10:623/udp` y la página que sirve `172.17.0.10:80`, dentro del rango autorizado `172.17.0.0/24`.
- **Qué hice:** releí localmente `pagina-172-17-0-10.html` («CTG · IPMI bajo exposición»); su pie de página cita «referencias públicas de CVE-2013-4786». Contrasté el CVE con NVD, FIRST EPSS y el catálogo CISA KEV, y lo relacioné con la enumeración IPMI de 19/11 en el mismo equipo.
- **Qué salió:** CVE-2013-4786 afecta a la autenticación RAKP de IPMI 2.0: permite obtener el resumen de la contraseña del BMC y adivinarla fuera de línea. Nmap identificó IPMI `2.0` en `172.17.0.10:623/udp`, así que la plataforma detectada coincide con la afectada. Datos: CVSS 7,5 alta (CVSS 3.0, NVD; C:H/I:N/A:N), EPSS 98,385 % (FIRST, dato del 22/11/2026) y no figura en CISA KEV (catálogo `2026.11.20`, consultado el 23/11/2026). Se añade como ficha **V-14** a `registro-vulns.md` y pasa a la posición 8 de la priorización de Parte 4 (las anteriores posiciones 8–13 pasan a 9–14).
- **Siguiente paso:** conservarlo como candidata a verificar administrativamente (fabricante/firmware del BMC y exposición de UDP 623), sin más pruebas de red ni captura de tráfico RAKP.
- **Limitación:** no se identificó producto ni firmware del BMC (sin CPE) ni se confirmó que RAKP esté habilitado; no se probó nada sobre el equipo.
- **Evidencias:** consultas EPSS/KEV en [epss-kev-ipmi-4786.txt](../evidencias/gonzalo/epss-kev-ipmi-4786.txt); enumeración IPMI en [enum-ipmi-172-17-0-10.nmap](../evidencias/gonzalo/enum-ipmi-172-17-0-10.nmap) y [XML](../evidencias/gonzalo/enum-ipmi-172-17-0-10.xml); página con la referencia en [pagina-172-17-0-10.html](../evidencias/gonzalo/pagina-172-17-0-10.html); ficha en [registro-vulns.md](../registro-vulns.md).

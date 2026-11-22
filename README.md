# Proyecto 2 · El informático fantasma (Mapping the Maze)

## 1. Datos del proyecto y autoría
- **Asignatura:** Hacking Ético (Curso de Especialización en Ciberseguridad)
- **Centro:** I.E.S. Rafael Alberti (Cádiz)
- **Estudiante / Auditora:** Natalia Alejo Pérez (Alumno 20)
- **Cliente:** Gadir Logística, S.L. (operador logístico del puerto de Cádiz)
- **Objetivo:** evaluación de vulnerabilidades (*Vulnerability Assessment*) no intrusiva de la red interna, para la renovación del ciberseguro corporativo.
- **Frontera legal y alcance autorizado:** `172.17.0.0/24` (especificado en [`scope.txt`](./scope.txt)).

---

## 2. Reglas de compromiso (RoE)
En cumplimiento del encargo del cliente y de los requisitos de la unidad:
1. **Solo identificación y enumeración:** no se ha ejecutado ninguna prueba de concepto, *exploit* ni intrusión.
2. **Cero fuerza bruta y sin prueba de credenciales:** no se han probado contraseñas, accesos por defecto ni sesiones en servicios que exigen autenticación. Solo se han usado sesiones **anónimas/nulas** y lectura de lo expuesto públicamente.
3. **Scripts de Nmap:** solo `ipmi-version` y `smb-os-discovery`, ambos de categoría `safe`, verificada con `nmap --script-help` antes de usarlos. Ninguno de las categorías `exploit`, `intrusive`, `dos`, `brute` ni `malware`.
4. **Protección de la disponibilidad:** escaneos con `-T3` y límite de paquetes por segundo (`--max-rate`), y `-T2` para el puerto 623. Tras los escaneos todos los equipos seguían activos.
5. **Respeto al alcance:** cualquier activo fuera de `172.17.0.0/24` (como `172.17.1.6`, de la hoja del cliente) se ha excluido y **no se ha escaneado**. `172.17.0.1` es la propia máquina de auditoría y no es objetivo.

---

## 3. Estructura del repositorio y enlaces a entregables

```
mapping-the-maze/
├── README.md                  # Este documento (presentación y enlaces)
├── scope.txt                  # Frontera legal de la auditoría (172.17.0.0/24)
├── inventario-cliente.csv     # Hoja de inventario entregada por el cliente (referencia)
├── cuadernos/
│   └── natalia-alejo.md       # Cuaderno con las cinco notas de la UD1 (Entradas 00 a 20)
├── inventario.md              # Equipos, puertos, servicios, cuentas y corrección al cliente
├── registro-vulns.md          # Registro priorizado: CVE, CWE, ATT&CK, CVSS, EPSS, KEV, evidencias y limitaciones
├── informe-cliente.md         # Informe ejecutivo final y respuesta a la noticia sobre OpenSSH
├── evidencias/
│   ├── 01_conexion_ssh_exitosa.png … 04_enum4linux_samba.jpg   # Capturas
│   ├── registro_sesion_terminal.log                            # Registro de sesión 1
│   └── salidas-laboratorio/                                    # Salidas de Nmap y registro de sesión 2
└── agente/                    # [Fase 5 · entrega posterior] src/, pruebas.md, traza.jsonl, informe.md
```
Además, `comparativa.md` (mano vs agente) se entregará en la raíz con la Fase 5.

### Acceso directo a los entregables
- 📋 **[Frontera legal (scope.txt)](./scope.txt):** subred `172.17.0.0/24`.
- 📓 **[Cuaderno de trabajo individual](./cuadernos/natalia-alejo.md):** trazabilidad con las cinco notas de la UD1, desde la conexión al laboratorio hasta la consulta de fuentes públicas.
- 🖥️ **[Inventario de equipos, servicios y cuentas](./inventario.md):** 8 equipos vivos con **mapa TCP completo** y UDP, enumeración por servicio, cuentas descubiertas sin autenticar y la corrección a la hoja del cliente (*qué sobra, qué falta y qué está mal*).
- 🛡️ **[Registro priorizado de vulnerabilidades](./registro-vulns.md):** ActiveMQ (CVE-2023-46604), Jenkins (CVE-2024-23897), Ghostcat en Tomcat (CVE-2020-1938), "sshole" en OpenSSH (CVE-2026-123158), IPMI (CVE-2013-4786), vsftpd (CVE-2011-2523), exposición en Samba y OpenSSH 7.7 (CVE-2018-15473), con CVSS, EPSS y KEV consultados el 2026-11-22.
- 📄 **[Informe para la dirección](./informe-cliente.md):** resumen ejecutivo, plan de tres medidas para el trimestre y respuesta a la noticia de OpenSSH.
- 📁 **[Evidencias](./evidencias/):** capturas, los registros de sesión y las salidas de cada escaneo ([`salidas-laboratorio/`](./evidencias/salidas-laboratorio/)). Los registros de sesión (`script`) permiten comprobar cada comando y su hora.

---

## 4. Nota sobre las evidencias

- **Única modificación de un registro bruto:** en `evidencias/registro_sesion_terminal.log` se sustituyó la **firma** del token JWT de `access.txt` por `[FIRMA-JWT-REDACTADA]`, para no publicar un token en el repositorio. El token estaba **caducado desde el 2024-10-05 21:32:53 UTC** y no se usó en ningún momento. El resto del log no se ha alterado.
- **Huella SHA256 del log original (conservado fuera del repositorio):** `c7654532c28eddfd3f41224ab144a749415f848ffbd72cb069494d20c03ab5fb`
- **Huella SHA256 del log publicado (con la firma redactada):** `5f941dde692b874edd4dbddd15c49de1238b92dbd74d95f1f121a3bd63649ad1`
- El fichero `access.txt` **no** está en el repositorio. Las demás salidas de `salidas-laboratorio/` se copiaron tal cual del laboratorio y no contienen el token.

---

## 5. Estado de las entregas
| Parte | Estado |
|---|---|
| 1 · Descubrir y mapear (`inventario.md`) | Entregable completo |
| 2 · Enumerar sin autenticar | Incluida en `inventario.md` |
| 3 y 4 · Registro priorizado e informe | Completos |
| 5 · El agente (`agente/`, `comparativa.md`) | **Pendiente** |

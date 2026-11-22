# Informe Ejecutivo de Evaluación de Vulnerabilidades

- **Destinatario:** Dirección y Gerencia — Gadir Logística, S.L. (Puerto de Cádiz)
- **Elaborado por:** Natalia Alejo Pérez (Equipo Consultor de Ciberseguridad)
- **Fecha:** 22 de noviembre de 2026
- **Motivo:** informe de vulnerabilidades priorizado para la renovación del ciberseguro
- **Alcance:** red interna `172.17.0.0/24`: **8 equipos activos**
- **Forma de trabajo:** solo se ha **mirado**. No se ha tocado, probado ni explotado nada, como pidió el cliente: nada de lo que se describe ha provocado cortes ni caídas.

---

## 1. Resumen para la Dirección

La hoja de inventario que dejó el antiguo informático **no refleja la realidad**: describe equipos Windows antiguos que no existen y omite servidores importantes. La red real son **8 equipos Linux**, y hemos reconstruido el inventario desde cero.

**Lo más importante:**

1. **Tres servidores tienen fallos graves, ya conocidos y explotados por ciberdelincuentes**, dos de ellos en ataques de *ransomware* (secuestro de datos):
   - el **servidor de mensajería** (`172.17.0.5`),
   - el **servidor de automatización Jenkins** (`172.17.0.6`),
   - el **servidor de aplicaciones Tomcat** (`172.17.0.4`).

   Lo sabemos por la **versión** del programa instalado, que coincide con las versiones afectadas. **No hemos comprobado explotándolos**, porque el acuerdo lo prohíbe. **Os avisamos de forma inmediata**: son los tres arreglos que recomendamos hacer primero.
2. **La consola con la que se gestionan los servidores** (`172.17.0.10`) es accesible desde toda la red interna. Es una puerta de administración de muy alto privilegio y debería estar aislada.
3. **Una carpeta compartida está abierta sin contraseña** (`172.17.0.8`): permite ver la lista de usuarios de la empresa y contiene un fichero con un código de acceso. Ese código **ya está caducado** (desde 2024), así que el riesgo inmediato es bajo, pero la carpeta debe cerrarse **hoy**, sin coste.
4. **Sobre la noticia "sshole":** de los 5 servidores con acceso SSH, **4 no deberían estar afectados** y **1 (`172.17.0.7`) no podemos descartarlo** todavía. Ver sección 3.

---

## 2. Qué recomendamos hacer primero

El trimestre permite pocas actuaciones, así que se ordenan por **riesgo real**: primero lo que los atacantes **ya están explotando** (catálogo CISA KEV) y, entre ellos, lo que se ha usado en *ransomware*.

### Las tres medidas del trimestre

#### Medida 1 · Actualizar el servidor de mensajería (`172.17.0.5`, Apache ActiveMQ)
- **Riesgo:** su versión (5.15.15) está afectada por un fallo (**CVE-2023-46604**) que permite a cualquiera con acceso de red **ejecutar órdenes en el servidor sin contraseña**. Está en el catálogo KEV y se ha usado en *ransomware*. Todo lo que dependa de la mensajería (operadores, envíos) quedaría expuesto.
- **Acción:** actualizar a una versión corregida y soportada. Mientras tanto, **limitar por cortafuegos** el acceso a los puertos de ActiveMQ (61616 y consola 8161) solo a los equipos que lo necesiten.
- **Esfuerzo:** medio, con ventana de mantenimiento (la mensajería se detiene al actualizar).

#### Medida 2 · Actualizar Jenkins (`172.17.0.6`)
- **Riesgo:** la versión 2.441 está afectada por **CVE-2024-23897**, que permite leer ficheros del servidor, incluidas claves. En KEV y usado en *ransomware*. *Matiz:* el servidor pide identificación en su página principal, lo que podría limitar el alcance; no se ha comprobado.
- **Acción:** actualizar a la versión **2.442 o superior** (o LTS 2.426.3 o superior). Si no se puede hoy, desactivar el acceso por línea de comandos y limitar el puerto 8080.
- **Esfuerzo:** bajo a medio.

#### Medida 3 · Cerrar el acceso AJP de Tomcat (`172.17.0.4`)
- **Riesgo:** la versión 9.0.30 tiene el fallo **Ghostcat (CVE-2020-1938)**: un atacante podría **descargar ficheros y configuraciones** de la aplicación. En KEV.
- **Acción:** actualizar Tomcat a la última versión 9.0.x y **desactivar el conector AJP** (puerto 8009) si no se usa o protegerlo con una clave.
- **Esfuerzo:** bajo (cambio de configuración).

### Acciones de coste cero para esta semana
| Acción | Equipo | Por qué |
|---|---|---|
| **Borrar `access.txt`, cerrar la carpeta `myshare` y exigir contraseña** en Samba | `172.17.0.8` | Deja de exponer usuarios y credenciales a toda la red |
| **Aislar la consola IPMI** con cortafuegos o red de gestión | `172.17.0.10` | Su protocolo tiene un fallo de diseño conocido (CVE-2013-4786) |
| **Restringir el SSH** a los equipos de administración | `172.17.0.7` y resto | Reduce la exposición mientras se aclara "sshole" |
| **Averiguar qué sistema es `172.17.0.7`** | `172.17.0.7` | Necesario para cerrar la duda de "sshole" |

> Quedan fuera del trimestre, por riesgo menor o por no estar confirmados, el servidor FTP antiguo (`172.17.0.3`, vsftpd 2.3.4, a retirar a medio plazo) y la actualización general de OpenSSH. Todo está detallado y justificado en el [registro de vulnerabilidades](registro-vulns.md).

---

## 3. Respuesta a la noticia sobre el fallo de OpenSSH ("sshole")

> **Consulta del cliente:** *«Acabo de ver en las noticias no sé qué fallo gordo que están aprovechando los atacantes, le llaman sshole. ¿Esto nos afecta? Juanma siempre decía que entraba a los servidores por SSH... Si hay que hacer algo, decidme qué, que el seguro nos lo va a preguntar.»*

### Respuesta corta
**Probablemente no os afecta en 4 de los 5 servidores con SSH. En 1 (`172.17.0.7`) no podemos asegurarlo todavía.** Hay que aclarar qué sistema es y, mientras tanto, limitar su acceso.

### Qué es el fallo
Es **CVE-2026-123158**, incluido por la agencia estadounidense CISA en su catálogo de fallos explotados el **9 de noviembre de 2026**. Permite **saltarse la contraseña** en el servidor SSH de **OpenSSH anterior a 10.6p2**, pero **solo en Linux con la librería musl y sin PAM** (típico de sistemas como Alpine). En esa situación, algunas cuentas aceptan **cualquier contraseña de entre 256 y 1024 caracteres**. La estimación de que se intente explotar en los próximos 30 días es del 95,5 %.

*Fuentes consultadas el 22/11/2026: catálogo CISA KEV (alta del 9/11/2026, que remite al aviso de Alpine Linux del 5/11/2026 y a las notas de versión de OpenSSH), ficha de NVD y estimación EPSS de FIRST.*

### Lo que podemos sostener con evidencias
Hay **5 servidores con SSH** (puerto 22):

| Equipo | Versión de OpenSSH | Sistema (por la propia versión) | ¿Afectado? |
|---|---|---|---|
| `172.17.0.4` | 7.9p1 | Debian 10 | No se espera |
| `172.17.0.6` | 9.2p1 | Debian 12 | No se espera |
| `172.17.0.8` | 9.6p1 | Ubuntu | No se espera |
| `172.17.0.9` | 9.2p1 | Debian 12 | No se espera |
| **`172.17.0.7`** | **7.7** | **No identificado** | **No se puede descartar** |

- **Todas las versiones son anteriores a 10.6p2**, pero eso **no basta**: el fallo exige un tipo concreto de sistema (musl, sin PAM).
- Los cuatro servidores marcados como Debian o Ubuntu usan componentes distintos (glibc y PAM), por eso **no se espera** que estén afectados. Es una **deducción por el nombre de la versión**, no una prueba.
- En `172.17.0.7`, la versión **no indica qué sistema** es. Hay sistemas como Alpine que tampoco lo indican, así que **no se puede descartar**.

### Lo que queda pendiente de comprobar, y por qué no se ha hecho
- **Identificar el sistema de `172.17.0.7`** (lo más sencillo: que lo confirme quien lo administra). Es la comprobación clave.
- **Probar si el fallo es real en cada servidor** exigiría intentar iniciar sesión con contraseñas largas, es decir, **probar contraseñas**, algo que el acuerdo prohíbe expresamente y que podría afectar a la operativa del puerto. **No se ha hecho.**

### Qué hacer
1. Averiguar el sistema de `172.17.0.7` y, si es Alpine u otro sistema con musl sin PAM, **actualizar OpenSSH a 10.6p2 o superior** de inmediato.
2. **Restringir por cortafuegos** el SSH de todos los servidores a los equipos de administración.
3. Mantener la actualización de OpenSSH en el mantenimiento programado.

### Qué responder a la aseguradora
> **Texto propuesto, para usar solo cuando las acciones estén realmente hechas** (sustituir lo marcado por [ ]):
>
> *"Gadir Logística ha inventariado todos sus servicios SSH (5 servidores) y ha contrastado sus versiones con el aviso CVE-2026-123158. Cuatro de ellos funcionan sobre Debian o Ubuntu, que no están afectados por las condiciones del fallo. El quinto ha sido identificado como [sistema confirmado] y [actualizado a OpenSSH 10.6p2 / no está afectado]. Además, el acceso SSH [ha sido restringido por cortafuegos a las IP de administración desde el (fecha)]."*

---

## 4. Qué no hemos hecho (y por qué)

Por las reglas pactadas (**solo mirar, sin tocar**):
- **No se ha explotado ni "probado" ningún fallo.** Los hallazgos graves se basan en la **versión** del programa. Confirmarlos es el trabajo de una **fase posterior y solo con vuestro permiso**.
- **No se han probado contraseñas** ni accesos por defecto (por ejemplo en las consolas web).
- **No se ha usado ninguna herramienta automática de vulnerabilidades.**
- **Equipos delicados:** los escaneos se hicieron con ritmo limitado y **ningún equipo se cayó**.

---

## 5. Próximos pasos
1. Presentar este informe a la aseguradora junto con el plan de tres medidas.
2. Ejecutar las **tres medidas** y las **acciones de coste cero** antes de que acabe el trimestre.
3. Autorizar, si se desea, una **fase de confirmación** de los fallos, con permiso escrito.
4. Auditoría de seguimiento a los 30 días para verificar el cierre.

---

## Anexo · Equipos y su situación

| Equipo | Qué es | Hallazgo principal | Prioridad |
|---|---|---|---|
| `172.17.0.5` | Mensajería (ActiveMQ) | CVE-2023-46604, ejecución remota | **1** |
| `172.17.0.6` | Jenkins | CVE-2024-23897, lectura de ficheros | **2** |
| `172.17.0.4` | Tomcat | CVE-2020-1938 (Ghostcat) | **3** |
| `172.17.0.7` | Servidor SSH | "sshole" (por aclarar) | 4 |
| `172.17.0.10` | Consola de gestión (IPMI) | Expuesta a toda la red | 5 |
| `172.17.0.3` | FTP antiguo | vsftpd 2.3.4 | 6 |
| `172.17.0.8` | Ficheros (Samba) | Carpeta abierta y usuarios visibles | 7 (coste 0) |
| `172.17.0.9` | Web de pruebas | Sin hallazgos relevantes | — |

Detalle técnico completo: [registro-vulns.md](registro-vulns.md) · Inventario: [inventario.md](inventario.md).

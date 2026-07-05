<p align="center">
  <img src="assets/icon-256.png" alt="Fragua" width="140" height="140">
</p>

<h1 align="center">Fragua</h1>

<p align="center">
  Escáner de seguridad ligero que analiza <b>objetivos web</b> y <b>máquinas</b>
  (Linux y Windows) y exporta los hallazgos a
  <a href="https://github.com/D3M0NH4NT3R/Crisol-RX">Crisol-RX</a>.
</p>

<p align="center">
  <sub>⚠️ <b>Proyecto en desarrollo — no está terminado.</b> Fragua es un escáner
  <b>preliminar</b>, no una herramienta de análisis avanzado. <b>No sustituye</b> a un
  pentest manual ni a herramientas dedicadas (Burp, ZAP, nmap, LinPEAS/WinPEAS), y
  <b>no garantiza</b> detectar todos los problemas de los apartados que revisa. Puede
  dar falsos positivos y negativos; confirma siempre los hallazgos a mano.</sub>
</p>

<p align="center">
  <img alt="Go" src="https://img.shields.io/badge/Go-1.21+-00ADD8?logo=go&logoColor=white">
  <img alt="Binario estático" src="https://img.shields.io/badge/binario-estático_único-15e6a0">
  <img alt="Sin CGO" src="https://img.shields.io/badge/CGO-off-8a52e8">
  <img alt="Estado" src="https://img.shields.io/badge/estado-en_desarrollo-e35d2a">
  <img alt="Uso ético" src="https://img.shields.io/badge/uso-ético-d6244a">
</p>

---

**Fragua** hace tres cosas, que puedes combinar:

1. **Web** (`--url`): configuración, cabeceras, TLS y superficie común de un sitio.
2. **Máquina — por fuera** (`--host`): escaneo de puertos/servicios (top 1000 de nmap).
3. **Máquina — por dentro** (`--host` + `--ssh-user`): auditoría de configuración y
   vías de escalada vía SSH, al estilo de **LinPEAS/WinPEAS** (Linux y Windows).

Todo **de forma no destructiva**, con salida **en vivo y coloreada**, y genera un
fichero JSON que subes tú a Crisol-RX. No sube nada por API: solo produce el archivo.

**Un único ejecutable estático** (sin CGO), para Windows, macOS (Apple Silicon e
Intel) y Linux, incluyendo ARM64. Usa solo la librería estándar de Go más
`golang.org/x/crypto/ssh` (pura-Go) para la auditoría por SSH.

> ⚠️ **Uso ético.** Audita únicamente sistemas para los que tengas autorización.

---

## Instalación

Descarga el binario de tu plataforma desde la
[página de releases](https://github.com/D3M0NH4NT3R/Fragua/releases). Es un
**único ejecutable**, sin instalación ni dependencias en tiempo de ejecución.

| Plataforma | Fichero |
|---|---|
| Windows (x64 / ARM64) | `fragua-windows-amd64.exe` · `fragua-windows-arm64.exe` |
| macOS (Apple Silicon / Intel) | `fragua-macos-apple-silicon` · `fragua-macos-intel` |
| Linux (x64 / ARM64) | `fragua-linux-amd64` · `fragua-linux-arm64` |

En macOS/Linux, dale permisos de ejecución la primera vez:

```bash
chmod +x fragua-*        # y ejecútalo con ./fragua-...
```

## Uso

```bash
# Web
./fragua --url https://objetivo.ejemplo

# Máquina: puertos/servicios "por fuera"
./fragua --host 10.0.0.5

# Máquina: por fuera + por dentro (auditoría SSH tipo PEAS)
./fragua --host 10.0.0.5 --ssh-user root --ssh-key ~/.ssh/id_ed25519

# Todo a la vez, en un solo reporte
./fragua --url https://objetivo.ejemplo --host 10.0.0.5 --ssh-user admin --ssh-pass '****'
```

Al arrancar muestra un banner con el icono y, **en vivo**, cada hallazgo que va
encontrando (con la severidad en color y, por dentro, por secciones tipo PEAS).
Al terminar escribe el JSON.

Luego, en Crisol-RX, sube el JSON con cualquiera de los dos:

- **Importar workspace** → crea un workspace nuevo con un proyecto y las vulnerabilidades.
- **Importar resultados de escaneo** (dentro de un proyecto) → añade los hallazgos al proyecto abierto.

### Opciones

**Objetivo** (indica `--url`, `--host` o ambos):

| Opción | Por defecto | Para qué |
|---|---|---|
| `--url` | — | Activo **web** a escanear. Si no pones esquema, se asume `https://`. |
| `--host` | — | **Máquina(s)** a auditar. Admite IP/host, varios con comas y **CIDR** (`10.0.0.0/24`). |
| `--targets` | — | Fichero con objetivos host (uno por línea; `#` = comentario). |

**Escaneo de host:**

| Opción | Por defecto | Para qué |
|---|---|---|
| `--ports` | top 1000 nmap | Puertos a escanear: `22,80,443` o rango `1-1024`. Vacío = los **1000 puertos top de nmap**. |
| `--all-ports` | `false` | Escanear los **65535** puertos (lento, más concurrencia). |
| `--no-portscan` | `false` | No escanear puertos (solo la auditoría SSH). |
| `--ssh-user` | — | Usuario SSH. **Activa la auditoría interna** (por dentro). |
| `--ssh-pass` | — | Contraseña SSH (o usa `--ssh-key`). |
| `--ssh-key` | — | Ruta a la clave privada SSH. |
| `--ssh-key-pass` | — | Passphrase de la clave privada (si está cifrada). |
| `--ssh-port` | `22` | Puerto SSH. |
| `--ssh-agent` | `true` | Usar ssh-agent (`SSH_AUTH_SOCK`) si está disponible. |
| `--ssh-known-hosts` | — | Ruta a `known_hosts` para verificar la clave del host. |
| `--ssh-strict` | `false` | Fallar si la clave del host no coincide con `known_hosts`. |

**General:**

| Opción | Por defecto | Para qué |
|---|---|---|
| `--lang` | `es` | Idioma del reporte: `es`, `en` o `both`. |
| `--out` | `crisol-<host>.json` | Fichero de salida (JSON de Crisol). |
| `--report` | — | Informe(s) humano(s) además del JSON: `md`, `html`, `csv`, `sarif` (admite lista: `md,html`). |
| `--name` | el host | Nombre del workspace/proyecto en el reporte. |
| `--scope` | — | Fichero de alcance: lista blanca de hosts/IP/CIDR; se omite todo lo que no esté dentro. |
| `--insecure` | `false` | No verificar el certificado TLS del objetivo web. |
| `--timeout` | `15` | Timeout por petición/conexión, en segundos. |
| `--max-time` | `0` | Tiempo máximo total del escaneo, en segundos (`0` = sin límite). |
| `--rate` | `0` | Máximo de peticiones web por segundo (`0` = sin límite). |
| `--max-pages` | `20` | Nº máximo de páginas a rastrear del mismo dominio (web). |
| `--no-crawl` | `false` | No rastrear enlaces; escanear solo la URL dada (web). |
| `--passive` | `false` | Modo pasivo: sin sondas activas (inyección, login, etc.). |
| `--quiet` | `false` | No mostrar los hallazgos en vivo (solo el resumen). |
| `--version` | `false` | Mostrar la versión y salir. |

> Puedes interrumpir el escaneo con **Ctrl-C** en cualquier momento: se guarda lo hallado hasta ese punto.

Variables de entorno: `NO_COLOR=1` desactiva el color de la salida.

### Ejemplos por opción

| Opción | Ejemplo |
|---|---|
| `--url` | `./fragua --url https://objetivo.ejemplo` |
| `--host` | `./fragua --host 10.0.0.5` |
| `--host` (CIDR/varios) | `./fragua --host 10.0.0.0/24,192.168.1.10` |
| `--targets` | `./fragua --targets hosts.txt` |
| `--ports` (lista) | `./fragua --host 10.0.0.5 --ports 22,80,443,8080` |
| `--ports` (rango) | `./fragua --host 10.0.0.5 --ports 1-1024` |
| `--all-ports` | `./fragua --host 10.0.0.5 --all-ports` |
| `--no-portscan` | `./fragua --host 10.0.0.5 --ssh-user root --ssh-key ~/.ssh/id_ed25519 --no-portscan` |
| `--ssh-user` + `--ssh-pass` | `./fragua --host 10.0.0.5 --ssh-user admin --ssh-pass 'Secreto123'` |
| `--ssh-key` | `./fragua --host 10.0.0.5 --ssh-user root --ssh-key ~/.ssh/id_ed25519` |
| `--ssh-key-pass` | `./fragua --host 10.0.0.5 --ssh-user root --ssh-key ~/.ssh/id_ed25519 --ssh-key-pass 'frase'` |
| `--ssh-port` | `./fragua --host 10.0.0.5 --ssh-user root --ssh-key ~/.ssh/id_ed25519 --ssh-port 2222` |
| `--ssh-known-hosts` | `./fragua --host 10.0.0.5 --ssh-user root --ssh-key k --ssh-known-hosts ~/.ssh/known_hosts --ssh-strict` |
| `--passive` | `./fragua --url https://objetivo.ejemplo --passive` |
| `--version` | `./fragua --version` |
| `--lang` (inglés) | `./fragua --url https://objetivo.ejemplo --lang en` |
| `--lang` (ambos) | `./fragua --url https://objetivo.ejemplo --lang both` |
| `--out` | `./fragua --url https://objetivo.ejemplo --out informe.json` |
| `--report` | `./fragua --url https://objetivo.ejemplo --report md,html` |
| `--report` (CI) | `./fragua --host 10.0.0.5 --report sarif` |
| `--name` | `./fragua --url https://objetivo.ejemplo --name "Cliente ACME"` |
| `--scope` | `./fragua --host 10.0.0.0/24 --scope alcance.txt` |
| `--max-time` | `./fragua --host 10.0.0.0/24 --max-time 600` |
| `--rate` | `./fragua --url https://objetivo.ejemplo --rate 20` |
| `--insecure` | `./fragua --url https://192.168.1.10 --insecure` |
| `--timeout` | `./fragua --url https://objetivo.ejemplo --timeout 30` |
| `--max-pages` | `./fragua --url https://objetivo.ejemplo --max-pages 50` |
| `--no-crawl` | `./fragua --url https://objetivo.ejemplo --no-crawl` |
| `--quiet` | `./fragua --url https://objetivo.ejemplo --quiet` |
| `NO_COLOR` (env) | `NO_COLOR=1 ./fragua --url https://objetivo.ejemplo` |

Ejemplo completo combinando web + host interno en un reporte bilingüe:

```bash
./fragua --url https://objetivo.ejemplo \
         --host 10.0.0.5 --ssh-user root --ssh-key ~/.ssh/id_ed25519 \
         --lang both --out informe.json
```

### Salida en vivo (verbose)

Por defecto Fragua es **verbose**: imprime cada hallazgo en cuanto lo detecta,
con un badge de severidad en color. La auditoría interna va además por **secciones
tipo PEAS** (`╔════╣ Sección`).

```
  [CRIT] Socket de Docker escribible  10.0.0.5
  [HIGH] Binarios SUID explotables (GTFOBins)  10.0.0.5
  [HIGH] Posible inyección SQL (basada en errores) en 'id'
  [MED ] Posible redirección abierta (open redirect) en 'next'
  [LOW ] Falta HSTS (Strict-Transport-Security)
  [INFO] Puerto abierto: 22/tcp (ssh)  10.0.0.5:22
```

`CRIT` rojo · `HIGH` naranja · `MED` amarillo · `LOW` verde · `INFO` morado.
Usa `--quiet` para silenciarlo, o `NO_COLOR=1` para quitar el color.

### Idioma del reporte

- Sin `--lang` → reporte en **español** (por defecto).
- `--lang en` → reporte en **inglés**.
- `--lang both` → genera **dos ficheros**, uno en español y otro en inglés, con
  sufijo de idioma (`crisol-<host>-es.json` y `crisol-<host>-en.json`; con
  `--out objetivo.json` salen `objetivo-es.json` y `objetivo-en.json`).

> En modo `both` el activo se escanea una vez por idioma. Si prefieres no duplicar
> el tráfico, genera cada idioma por separado con `--lang es` / `--lang en`.

## Qué comprueba

Todo son comprobaciones pasivas y activas ligeras, **no destructivas** y de solo
lectura.

### Web (`--url`)

Rastrea el mismo dominio (hasta `--max-pages`) y repite las comprobaciones por página.

- **Transporte/TLS**: HTTP en claro, HTTP que no redirige a HTTPS, certificado
  caducado/autofirmado/cadena incompleta/host no cubierto/firma débil/clave corta,
  TLS 1.0/1.1 y cifrados débiles.
- **Cabeceras**: CSP ausente o débil, anti-clickjacking, nosniff, Referrer-Policy,
  Permissions-Policy, HSTS, COOP/CORP, X-XSS-Protection, divulgación de tecnología.
- **Configuración/superficie**: cookies sin flags, CORS mal configurado, métodos
  HTTP peligrosos, Basic sobre HTTP, listado de directorios, `crossdomain.xml`,
  ficheros expuestos (`.git`, `.env`, `wp-config.php.bak`, `.git-credentials`,
  `swagger.json`, `actuator/env`…), `robots.txt`, detección de stack.
- **Por página**: posible **XSS reflejado**, posible **SQLi basada en errores**,
  contenido mixto, formulario de credenciales sin cifrar.
- **Avanzadas** (activas ligeras): **open redirect**, **CORS por reflexión de
  Origin**, **Host header injection**, **GraphQL con introspección**, **secretos en
  HTML/JS** (AWS, GitHub, Google, Slack, Stripe, claves privadas…).

### Máquina por fuera (`--host`)

Escaneo **TCP connect** de los **1000 puertos top de nmap** (o `--ports` /
`--all-ports`), estilo **`-Pn`** (sin ping ICMP), con detección de servicio/banner.

- Un puerto abierto se registra como **`info`** (reconocimiento). **No** es una
  vulnerabilidad por el mero hecho de estar abierto.
- Solo se generan vulnerabilidades cuando se **detecta un problema real** (estilo `nmap -A`):
  - **Protocolos en texto claro**: Telnet, r-services (rexec/rlogin/rsh).
  - **Versiones vulnerables conocidas** por banner (p. ej. vsFTPd 2.3.4 backdoor,
    ProFTPD 1.3.3c, OpenSSH muy antiguo).
  - **Servicios sin autenticación**, verificados con una sonda ligera: FTP con
    login anónimo, Redis y Memcached accesibles sin contraseña.
  - **SNMP con comunidad `public`** (UDP/161) y **SMB sin firma obligatoria**
    (SMB2 negotiate), comprobados por red sin credenciales.
  - **SMBv1 habilitado** (sonda SMB1 negotiate) → expone la familia **EternalBlue /
    MS17-010** (CVE-2017-0143…0148). Detectado por red, sin credenciales.
  - **Pistas de CVE por versión** del banner/cabecera `Server`: Apache 2.4.49/50,
    nginx 1.3/1.4.0, IIS 6.0/7.5, ProFTPD 1.3.3c/1.3.5, OpenSSL 0.9.8/1.0.1
    (Heartbleed), Exim 4.87–4.91, **Webmin 1.890–1.920 (CVE-2019-15107)**,
    **Samba 3.0.20 (CVE-2007-2447) / 3.5.0 (SambaCry)**, UnrealIRCd 3.2.8.1, PHP 5.x…

### Máquina por dentro (`--host` + `--ssh-user`) — estilo PEAS

Se autentica por SSH (contraseña o clave), detecta el SO y enumera debilidades y
vías de escalada, por **secciones coloreadas**.

**Linux**: sistema y kernel + **sugeridor de exploits de kernel** (DirtyCow,
DirtyPipe, Sequoia…), usuarios con shell, cuentas UID 0 extra, contraseñas vacías,
`sshd_config`, permisos de `/etc/shadow`, cortafuegos, servicios a la escucha,
actualizaciones pendientes, **SUID/SGID explotables (GTFOBins)**, world-writable,
**directorios del PATH escribibles**, **sudo (NOPASSWD / versión vulnerable)**,
**`/etc/passwd` escribible**, **socket de Docker escribible**, **capabilities
peligrosas**, **grupos con privilegios (docker/lxd/disk…)**, **NFS `no_root_squash`**,
**cron/systemd escribibles**, **claves SSH con permisos laxos**, **secretos en
entorno/fstab/historial**, **ASLR**, **versiones de software** relevante y
**configuración de Samba** (recursos escribibles con `wide links`, `force user` o
acceso invitado → *symlink escape* y plantado de ficheros como otro usuario, p. ej.
`~/.ssh/authorized_keys`).

**Windows** (requiere **OpenSSH Server** activo; usa PowerShell): sistema,
Administradores, cortafuegos, Defender, SMBv1, UAC, **servicios con ruta sin
comillas**, parches, **privilegios de token (SeImpersonate…)**, **AlwaysInstallElevated**,
**autologon en el registro**, **cmdkey**, **ficheros de instalación desatendida con
contraseña**, **tareas programadas / autoruns**, **historial de PowerShell**,
**sesiones PuTTY**, **LSASS sin RunAsPPL**, **recursos compartidos** y **software
instalado**.

> **No es un PEAS 1:1** (PEAS tiene cientos de comprobaciones), pero cubre los
> vectores de escalada más habituales con la misma filosofía y salida coloreada, y
> solo marca como vulnerabilidad lo realmente accionable. Las comprobaciones son de
> **solo lectura**: no cambian la configuración del sistema. La clave del host SSH
> se acepta automáticamente (no se verifica), apropiado para una auditoría.

## Formato de salida

El fichero es el formato `crisol-workspace` (el mismo de «Exportar workspace» de
Crisol-RX), por eso Crisol lo importa tal cual.

## Estado del proyecto

Fragua está **en desarrollo activo**. Faltan comprobaciones por añadir, la
cobertura no es completa y la interfaz/salida pueden cambiar entre versiones. Se
agradecen ideas y reportes de fallos en los *issues*.

## Uso ético y responsabilidad

Fragua realiza peticiones y conexiones a objetivos remotos, incluyendo
comprobaciones activas (open redirect, Host falso, comilla SQL, introspección
GraphQL, escaneo de puertos, login SSH). **Úsalo solo contra sistemas para los que
tengas autorización explícita.** El autor no se responsabiliza de un uso indebido.

## Licencia

Ver [LICENSE](LICENSE). Atribución al autor original obligatoria.

---

<p align="center">
  Hecho por <a href="https://github.com/D3M0NH4NT3R">D3M0NH4NT3R</a> · complemento de
  <a href="https://github.com/D3M0NH4NT3R/Crisol-RX">Crisol-RX</a>
</p>

<p align="center">
  <img src="assets/icon-256.png" alt="Fragua" width="140" height="140">
</p>

<h1 align="center">Fragua</h1>

<p align="center">
  Escáner web ligero que detecta vulnerabilidades comunes y las exporta a
  <a href="https://github.com/D3M0NH4NT3R/Crisol-RX">Crisol-RX</a>.
</p>

<p align="center">
  <sub>⚠️ <b>Proyecto en desarrollo — no está terminado.</b> Fragua es un escáner
  <b>preliminar</b>, no una herramienta de análisis avanzado: hace una primera pasada
  rápida sobre configuración, cabeceras, TLS y superficie común. <b>No sustituye</b> a un
  pentest manual ni a un DAST completo (Burp, ZAP), y <b>no garantiza</b> detectar todos
  los problemas de los apartados que revisa. Puede dar falsos positivos y negativos;
  confirma siempre los hallazgos a mano.</sub>
</p>

<p align="center">
  <img alt="Go" src="https://img.shields.io/badge/Go-stdlib_only-00ADD8?logo=go&logoColor=white">
  <img alt="Sin dependencias" src="https://img.shields.io/badge/dependencias-0-15e6a0">
  <img alt="Binario único" src="https://img.shields.io/badge/binario-único-8a52e8">
  <img alt="Estado" src="https://img.shields.io/badge/estado-en_desarrollo-e35d2a">
  <img alt="Uso ético" src="https://img.shields.io/badge/uso-ético-d6244a">
</p>

---

**Fragua** escanea un activo web, comprueba fallos de configuración, cabeceras,
TLS y superficie común (**de forma segura y no destructiva**) y **genera un
fichero JSON** que subes tú a Crisol-RX. No sube nada por API: solo produce el
archivo.

Misma filosofía que Crisol-RX: **sin dependencias** y **un único ejecutable**.
Disponible para Windows, macOS (Apple Silicon e Intel) y Linux, incluyendo ARM64.

> ⚠️ **Uso ético.** Escanea únicamente activos para los que tengas autorización.

---

## Instalación

Descarga el binario de tu plataforma desde la
[página de releases](https://github.com/D3M0NH4NT3R/Fragua/releases). Es un
**único ejecutable**, sin instalación ni dependencias.

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
./fragua --url https://objetivo.ejemplo --out objetivo.json
```

Al arrancar muestra un banner con el icono y, **en vivo**, cada vulnerabilidad
que va encontrando (con la severidad en color). Al terminar escribe el JSON.

Luego, en Crisol-RX, sube el JSON con cualquiera de los dos:

- **Importar workspace** → crea un workspace nuevo con un proyecto y las vulnerabilidades.
- **Importar resultados de escaneo** (dentro de un proyecto) → añade los hallazgos al proyecto abierto.

### Opciones

| Opción | Por defecto | Para qué |
|---|---|---|
| `--url` | — | Activo a escanear (**obligatorio**). Si no pones esquema, se asume `https://`. |
| `--lang` | `es` | Idioma del reporte: `es`, `en` o `both`. |
| `--out` | `crisol-<host>.json` | Fichero de salida. |
| `--name` | el host | Nombre del workspace/proyecto en el reporte. |
| `--insecure` | `false` | No verificar el certificado TLS del objetivo. |
| `--timeout` | `15` | Timeout por petición, en segundos. |
| `--max-pages` | `20` | Nº máximo de páginas a rastrear del mismo dominio. |
| `--no-crawl` | `false` | No rastrear enlaces; escanear solo la URL dada. |
| `--quiet` | `false` | No mostrar los hallazgos en vivo (solo el resumen). |

Variables de entorno: `NO_COLOR=1` desactiva el color de la salida.

### Salida en vivo (verbose)

Por defecto Fragua es **verbose**: imprime cada hallazgo en cuanto lo detecta,
con un badge de severidad en color.

```
  [CRIT] Fichero .env expuesto  https://host/.env
  [HIGH] Posible inyección SQL (basada en errores) en 'id'
  [HIGH] CORS refleja el Origin de la petición (con credenciales)
  [MED ] Posible redirección abierta (open redirect) en 'next'
  [LOW ] Falta HSTS (Strict-Transport-Security)
  [INFO] Tecnologías detectadas en el activo
```

`CRIT` rojo · `HIGH` naranja · `MED` amarillo · `LOW` verde · `INFO` morado.
Usa `--quiet` para silenciarlo.

### Idioma del reporte

- Sin `--lang` → reporte en **español** (por defecto).
- `--lang en` → reporte en **inglés**.
- `--lang both` → genera **dos ficheros**, uno en español y otro en inglés, con
  sufijo de idioma (`crisol-<host>-es.json` y `crisol-<host>-en.json`; con
  `--out objetivo.json` salen `objetivo-es.json` y `objetivo-en.json`).

> En modo `both` el activo se escanea una vez por idioma. Si prefieres no duplicar
> el tráfico, genera cada idioma por separado con `--lang es` / `--lang en`.

## Qué comprueba

Comprobaciones pasivas y activas ligeras, todas **GET/OPTIONS/POST/TRACE y no
destructivas**. Por defecto **rastrea el mismo dominio** (hasta `--max-pages`) y
repite las comprobaciones por página.

**Transporte y TLS**
- Comunicación sin cifrar (HTTP) y HTTP que no redirige a HTTPS.
- Certificado: caducado, próximo a caducar, autofirmado o cadena incompleta,
  host no cubierto (CN/SAN), firma débil (SHA-1/MD5) y clave RSA corta.
- Protocolos obsoletos soportados (TLS 1.0 / 1.1) y suites de cifrado débiles.

**Cabeceras**
- CSP ausente **o débil** (`unsafe-inline`, `unsafe-eval`, comodines).
- Anti-clickjacking, X-Content-Type-Options, Referrer-Policy, Permissions-Policy.
- HSTS ausente o débil; COOP y CORP; X-XSS-Protection deshabilitada.
- Divulgación de tecnología (Server, X-Powered-By, generator…).

**Configuración y superficie**
- Cookies sin Secure / HttpOnly / SameSite.
- CORS mal configurado (comodín con credenciales).
- Métodos HTTP peligrosos (PUT/DELETE/PATCH/CONNECT/TRACE/TRACK) y método TRACE (XST).
- Autenticación Basic sobre HTTP.
- Listado de directorios habilitado.
- Política de dominio cruzado permisiva (`crossdomain.xml`, `clientaccesspolicy.xml`).
- Ficheros sensibles expuestos: `.git`, `.env`, `.env.local`, `.svn`, `web.config`,
  `wp-config.php.bak`, `.htpasswd`, `.git-credentials`, `phpinfo.php`,
  `.aws/credentials`, `docker-compose.yml`, `backup.sql`, `actuator/health`,
  `actuator/env`, `appsettings.json`, `WEB-INF/web.xml`, `server-status`,
  `security.txt`, y más.
- `robots.txt` que revela rutas internas y detección de stack (WordPress/Drupal/Joomla).

**Fugas de información (en el cuerpo)**
- Clave privada (PEM) servida, trazas de error / SQL, IPs internas y correos.

**Por página**
- Posible **XSS reflejado** (marcador benigno; requiere confirmación manual).
- Posible **inyección SQL basada en errores** (comilla en parámetros reales).
- **Contenido mixto** (recurso HTTP en página HTTPS).
- **Formulario de credenciales sin cifrar**.

**Comprobaciones avanzadas** (activas ligeras, no destructivas)
- **Open redirect**: parámetros de redirección que apuntan a dominios externos.
- **CORS por reflexión de Origin** (peor si permite credenciales).
- **Inyección de cabecera Host** (envenenamiento de caché / de enlaces).
- **GraphQL con introspección** abierta.
- **Secretos/credenciales** embebidos en HTML o ficheros JavaScript (claves AWS,
  tokens de GitHub/Google/Slack/Stripe, claves privadas…).
- Documentación de API expuesta (`swagger.json`, `openapi.json`, `api-docs`).

No es un DAST completo (no reemplaza a Burp/ZAP ni prueba SQLi a fondo o lógica de
negocio). Cubre configuración, cabeceras, TLS y superficie común, y deja los
hallazgos listos para revisar y ampliar en Crisol-RX. Los hallazgos repetidos se
deduplican por título + activo.

## Formato de salida

El fichero es el formato `crisol-workspace` (el mismo de «Exportar workspace» de
Crisol-RX), por eso Crisol lo importa tal cual.

## Estado del proyecto

Fragua está **en desarrollo activo**. Faltan comprobaciones por añadir, la
cobertura no es completa y la interfaz/salida pueden cambiar entre versiones.
Se agradecen ideas y reportes de fallos en los *issues*.

## Uso ético y responsabilidad

Fragua realiza peticiones a un objetivo remoto, incluyendo algunas comprobaciones
activas (open redirect, Host falso, comilla SQL, introspección GraphQL).
**Escanéalo solo si tienes autorización explícita.** El autor no se responsabiliza
de un uso indebido.

## Licencia

Ver [LICENSE](LICENSE). Atribución al autor original obligatoria.

---

<p align="center">
  Hecho por <a href="https://github.com/D3M0NH4NT3R">D3M0NH4NT3R</a> · complemento de
  <a href="https://github.com/D3M0NH4NT3R/Crisol-RX">Crisol-RX</a>
</p>

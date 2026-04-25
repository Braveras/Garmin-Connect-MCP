# Garmin MCP Server

MCP server que conecta Garmin Connect con Claude y otros clientes MCP compatibles, exponiendo tus datos de fitness y salud.

> **Fork de [Taxuspt/garmin_mcp](https://github.com/Taxuspt/garmin_mcp) con el fix de autenticación aplicado** ([PR #84](https://github.com/Taxuspt/garmin_mcp/pull/84)):  
> Migrado de la API `garth` (obsoleta) a `garminconnect==0.3.2`, resolviendo los errores de login que afectaban al repo original.

Garmin Connect se accede mediante la librería [python-garminconnect](https://github.com/cyberjunky/python-garminconnect).

## Herramientas disponibles (96+)

- ✅ Gestión de actividades (14 herramientas)
- ✅ Salud y bienestar (31 herramientas) — pasos, sueño, frecuencia cardíaca, estrés, SpO2...
- ✅ Entrenamiento y rendimiento (9 herramientas)
- ✅ Entrenamientos/Workouts (8 herramientas)
- ✅ Dispositivos (7 herramientas)
- ✅ Equipamiento (5 herramientas)
- ✅ Control de peso (5 herramientas)
- ✅ Retos y medallas (10 herramientas)
- ✅ Nutrición (8 herramientas)
- ✅ Salud femenina (3 herramientas)
- ✅ Perfil de usuario (3 herramientas)

## Instalación

### Requisitos previos

- Python 3.10+
- `uv` instalado ([instrucciones](https://docs.astral.sh/uv/getting-started/installation/))
- Cuenta de Garmin Connect

### Paso 1: Autenticación con Garmin (una sola vez)

Antes de usar el servidor MCP, debes autenticarte desde la terminal para guardar los tokens OAuth:

```bash
uvx --python 3.12 --from git+https://github.com/Braveras/Garmin-Connect-MCP garmin-mcp-auth
```

Se te pedirá email, contraseña y código MFA (si lo tienes activado). Los tokens se guardan en `~/.garminconnect` y son válidos aproximadamente 6 meses.

**Opciones adicionales:**

```bash
# Usar variables de entorno para las credenciales
GARMIN_EMAIL=tu@email.com GARMIN_PASSWORD=tu_password garmin-mcp-auth

# Verificar que los tokens actuales funcionan
garmin-mcp-auth --verify

# Forzar re-autenticación (cuando caduquen los tokens)
garmin-mcp-auth --force-reauth

# Usar ruta personalizada para los tokens
garmin-mcp-auth --token-path ~/.garmin_tokens
```

### Paso 2: Configurar el cliente MCP

#### Claude Code CLI (terminal)

```bash
claude mcp add garmin -- uvx --python 3.12 --from git+https://github.com/Braveras/Garmin-Connect-MCP garmin-mcp
```

#### Claude Desktop

Edita el fichero de configuración:

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "garmin": {
      "command": "uvx",
      "args": [
        "--python",
        "3.12",
        "--from",
        "git+https://github.com/Braveras/Garmin-Connect-MCP",
        "garmin-mcp"
      ]
    }
  }
}
```

> No es necesario incluir `GARMIN_EMAIL` ni `GARMIN_PASSWORD` — el servidor usa los tokens guardados en el paso 1.

Reinicia Claude Desktop y listo.

#### Desde una copia local del repositorio

```bash
git clone https://github.com/Braveras/Garmin-Connect-MCP
cd Garmin-Connect-MCP
uv sync
```

Configura el cliente MCP apuntando al directorio local:

```json
{
  "mcpServers": {
    "garmin-local": {
      "command": "uv",
      "args": [
        "--directory",
        "/ruta/completa/a/Garmin-Connect-MCP",
        "run",
        "garmin-mcp"
      ]
    }
  }
}
```

## Configuración

Variables de entorno disponibles:

| Variable | Descripción |
|---|---|
| `GARMIN_EMAIL` | Tu email de Garmin Connect |
| `GARMIN_EMAIL_FILE` | Ruta a un fichero con tu email |
| `GARMIN_PASSWORD` | Tu contraseña de Garmin Connect |
| `GARMIN_PASSWORD_FILE` | Ruta a un fichero con tu contraseña |
| `GARMIN_IS_CN` | `true` para usar Garmin Connect China (garmin.cn) |
| `GARMINTOKENS` | Ruta personalizada para los tokens OAuth (por defecto `~/.garminconnect`) |

> No puedes usar `GARMIN_EMAIL` y `GARMIN_EMAIL_FILE` simultáneamente.

### Garmin Connect China

```bash
GARMIN_IS_CN=true garmin-mcp-auth
# o
garmin-mcp-auth --is-cn
```

## Ejemplos de uso

Una vez conectado en Claude, puedes preguntar:

- *"¿Cuántos pasos hice ayer?"*
- *"¿Cómo dormí la semana pasada?"*
- *"Muéstrame mis últimas 5 actividades"*
- *"¿Cuál es mi frecuencia cardíaca en reposo hoy?"*
- *"¿Cómo está mi estado de entrenamiento?"*

## Solución de problemas

### Error de autenticación / tokens caducados

```bash
garmin-mcp-auth --force-reauth
```

### `uvx` no se encuentra (Claude Desktop)

Claude Desktop no siempre hereda el PATH del sistema. Usa la ruta completa:

```bash
which uvx   # en macOS/Linux
where uvx   # en Windows
```

Y úsala en la configuración:

```json
{
  "mcpServers": {
    "garmin": {
      "command": "/ruta/completa/a/uvx",
      "args": ["--python", "3.12", "--from", "git+https://github.com/Braveras/Garmin-Connect-MCP", "garmin-mcp"]
    }
  }
}
```

### MFA requerido pero sin terminal disponible

Si el MCP server arranca sin terminal (ej. desde Claude Desktop) y los tokens han caducado, verás un error. Solución:

1. Abre una terminal
2. Ejecuta `garmin-mcp-auth`
3. Introduce el código MFA
4. Reinicia Claude Desktop

### Logs

- **macOS:** `~/Library/Logs/Claude/mcp-server-garmin.log`
- **Windows:** `%APPDATA%\Claude\logs\mcp-server-garmin.log`

## Tests

```bash
# Tests de integración (con API mockeada)
uv run pytest tests/integration/ -v

# Tests unitarios
uv run pytest tests/unit/ -v

# Tests end-to-end (requieren credenciales reales)
uv run pytest tests/e2e/ -m e2e -v
```

## Créditos

- Repositorio original: [Taxuspt/garmin_mcp](https://github.com/Taxuspt/garmin_mcp)
- Fix de autenticación: [MaxLazar (PR #84)](https://github.com/Taxuspt/garmin_mcp/pull/84)
- Librería Garmin Connect: [cyberjunky/python-garminconnect](https://github.com/cyberjunky/python-garminconnect)

## Licencia

MIT — ver [LICENSE](LICENSE)

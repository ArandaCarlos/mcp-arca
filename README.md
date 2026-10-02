# MCP ARCA — Facturá en ARCA hablándole a la IA

Servidor MCP que conecta Claude con el web service de factura electrónica de ARCA (WSFEv1). Le pedís una factura en lenguaje natural y la emite.

📺 Video completo: https://youtu.be/6PQTRJ_INLk?si=oO0QveWzgenr68PN

> ⚠️ Este proyecto está pensado para el entorno de **homologación** (prueba) de ARCA. Las facturas que emitas ahí no tienen validez fiscal.

---

## Herramientas que expone

| Herramienta | Qué hace |
|---|---|
| `consultar_ultimo_comprobante` | Devuelve el número del último comprobante autorizado |
| `emitir_factura` | Emite una factura y devuelve número, CAE y vencimiento |
| `consultar_comprobante` | Trae los datos completos de una factura ya emitida |

## Requisitos

- Node.js 22 o superior
- Certificado de homologación de ARCA con el servicio `wsfe` autorizado ([cómo sacarlo](link al video del certificado)) : https://www.youtube.com/watch?v=LcAI7eE_ZRs&t=1986s
- Claude Code para construirlo y Claude Desktop para usarlo

## Instalación

```bash
git clone https://github.com/TU_USUARIO/mcp-arca.git
cd mcp-arca
npm install
```

Creá un archivo `.env` en la raíz:

```env
ARCA_CUIT=20XXXXXXXXX
ARCA_CERT_PATH=/ruta/absoluta/certificado.crt
ARCA_KEY_PATH=/ruta/absoluta/clave.key
```

## Conectarlo

**Claude Code:**

```bash
claude mcp add arca --scope user -- node --use-system-ca /ruta/absoluta/mcp-arca/src/server.js
```

**Claude Desktop:** agregá esto a `claude_desktop_config.json`
(Windows: `%APPDATA%\Claude\`, macOS: `~/Library/Application Support/Claude/`)

```json
{
  "mcpServers": {
    "arca": {
      "command": "node",
      "args": ["--use-system-ca", "C:/ruta/absoluta/mcp-arca/src/server.js"]
    }
  }
}
```

En Windows usá barras `/` en la ruta, no `\`. Después cerrá Claude Desktop por completo (desde la bandeja del sistema) y volvé a abrirlo.

## Probalo

- "¿Cuál fue mi último comprobante emitido?"
- "Haceme una factura C a consumidor final por $50.000 por servicio de desarrollo web"
- "Mostrame la factura que acabamos de emitir"

---

## Los prompts del video

### 1. Construir el servidor

```
Quiero crear un servidor MCP que permita a Claude emitir facturas electrónicas en ARCA (ex AFIP) usando el web service de factura electrónica (WSFEv1), en el entorno de HOMOLOGACIÓN.

Contexto: ya tengo código funcionando que se autentica con WSAA y emite facturas, con certificado y clave privada. Está en [RUTA DE TU PROYECTO]. Revisalo primero y reutilizá esa lógica en vez de reescribirla.

Requisitos:
- Usar el SDK oficial de MCP, con transporte stdio para correrlo local.
- Dos herramientas:
  1. consultar_ultimo_comprobante(punto_venta, tipo_comprobante)
  2. emitir_factura(tipo_comprobante, punto_venta, concepto, doc_tipo, doc_nro, importe_total, descripcion): obtiene el próximo número, solicita el CAE y devuelve número, CAE y vencimiento.
- Descripciones de herramientas claras y en español, con los valores válidos (ej: factura C = tipo 11, consumidor final = doc_tipo 99 con doc_nro 0).
- Validar parámetros antes de llamar a ARCA y devolver errores legibles.
- CUIT y rutas del certificado por variables de entorno. Nunca hardcodeadas ni logueadas.
- Forzar homologación: apuntar a producción requiere una variable de entorno explícita.
- Cachear el token de WSAA hasta que venza.
- README con cómo registrarlo en Claude Code y en Claude Desktop.

Al final, probá ambas herramientas contra homologación y mostrame el resultado.
```

### 2. Agregar la consulta de comprobantes

```
Agregá una tercera herramienta al servidor MCP de ARCA: consultar_comprobante.

- Recibe tipo_comprobante, punto_venta y numero_comprobante.
- Llama a FECompConsultar de WSFEv1 (homologación), reutilizando la autenticación y el token cacheado.
- Devuelve los datos en formato legible: tipo, punto de venta, número, fecha, receptor, importe total, CAE, vencimiento del CAE y resultado.
- Si el comprobante no existe, devolvé un mensaje claro en vez del error crudo de ARCA.

La descripción debe aclarar que sirve para verificar una factura ya emitida, y que si el usuario dice "la última factura" o "la que acabamos de hacer", primero se puede usar consultar_ultimo_comprobante.

No modifiques las otras herramientas. Al terminar, probala consultando el último comprobante emitido.
```

---

## Seguridad

- El certificado y la clave privada nunca salen de tu máquina. La IA no los ve: solo llama a las herramientas.
- No subas `.env`, `.crt` ni `.key` a ningún repositorio.
- Si lo pasás a producción, agregá un paso de confirmación antes de emitir: una factura emitida no se borra, se anula con nota de crédito.

---

Hecho por **char.developer** · [YouTube](link) · [TikTok](https://tiktok.com/@char.developer) · [chardeveloper.com.ar](https://chardeveloper.com.ar)

---
name: CSS to Figma Tokens Converter
description: Transforma variables CSS a formato DTCG (.tokens.json) compatible con plugins de importación de Figma
---

# CSS to Figma Tokens Converter

Esta skill convierte variables CSS (custom properties) al formato Design Tokens Community Group (DTCG) optimizado para importación en Figma mediante plugins como "Variables Import".

## Formato de Entrada Esperado

Variables CSS definidas con `--` en archivos `.css`:

```css
:root {
  /* Colores */
  --color-primary: #C8A96E;
  --color-secondary: #E8D5B0;
  
  /* Tipografía */
  --font-sans: 'DM Sans', system-ui, sans-serif;
  --font-size-base: 16px;
  
  /* Espaciado */
  --spacing-sm: 8px;
  --spacing-md: 16px;
}
```

## Formato de Salida (DTCG para Figma)

El formato final debe cumplir estas reglas:

1. **Usar `$value` y `$type`** en cada token
2. **Tipos soportados por Figma Variables**:
   - `color` - Para colores hexadecimales, rgb, hsl
   - `number` - Para valores numéricos puros (sin unidades)
   - `string` - Para texto, fuentes, etc.
   - ⚠️ NO usar `fontFamily`, `dimension` u otros tipos del estándar DTCG completo

3. **Estructura jerárquica** usando objetos anidados
4. **Nombres limpios** sin prefijos `--` ni guiones innecesarios

### Ejemplo de Salida Correcta

```json
{
  "brand": {
    "primary": {
      "$value": "#C8A96E",
      "$type": "color"
    }
  },
  "typography": {
    "family": {
      "sans": {
        "$value": "DM Sans, system-ui, sans-serif",
        "$type": "string"
      }
    },
    "size": {
      "base": {
        "$value": 16,
        "$type": "number"
      }
    }
  }
}
```

## Reglas de Transformación

### 1. Detección de Tipos

- **Color**: Si el valor es `#HEX`, `rgb()`, `rgba()`, `hsl()`, `hsla()` → `$type: "color"`
- **Número**: Si el valor es numérico con `px`, `rem`, `em` → extraer número → `$type: "number"`
- **String**: Todo lo demás (fuentes, texto) → `$type: "string"`

### 2. Normalización de Nombres

Convertir nombres de variables CSS a rutas jerárquicas:

| CSS Variable | Ruta DTCG |
|--------------|-----------|
| `--color-brand-primary` | `color.brand.primary` |
| `--font-family-sans` | `typography.family.sans` |
| `--spacing-md` | `spacing.md` |
| `--text-color-muted` | `text.color.muted` |

### 3. Limpieza de Valores

- **Colores**: Mantener formato hexadecimal en mayúsculas: `#C8A96E`
- **Números**: Extraer solo el valor numérico: `16px` → `16`
- **Fuentes**: Limpiar comillas si es necesario, mantener fallbacks

### 4. Agrupación por Categorías

Crear archivos separados para facilitar la importación en colecciones:

- `colors.tokens.json` - Solo tokens de color
- `typography.tokens.json` - Familias de fuentes (string)
- `sizes.tokens.json` - Tamaños numéricos
- `spacing.tokens.json` - Espaciados (opcional)

## Proceso de Conversión

1. **Leer el archivo CSS** y extraer todas las variables `--*`
2. **Clasificar por tipo** (color, número, string)
3. **Parsear nombres** y crear estructura jerárquica
4. **Generar archivos JSON** separados por categoría
5. **Validar** que todos los tokens tengan `$value` y `$type`

## Ejemplo de Uso

```
USER: Convierte @[theme.css] a tokens de Figma
ASSISTANT: [Lee el archivo CSS]
ASSISTANT: [Crea colors.tokens.json, typography.tokens.json, sizes.tokens.json]
ASSISTANT: He creado 3 archivos de tokens listos para importar en Figma...
```

## Limitaciones Conocidas

- Figma Variables NO soporta:
  - Gradientes (usar como string o dividir en colores individuales)
  - Sombras complejas (usar como string)
  - Tipos `fontFamily`, `dimension`, `duration` del estándar DTCG completo
  
- El plugin puede fallar si:
  - Hay tipos desconocidos
  - Los valores tienen formato incorrecto
  - Falta `$value` o `$type` en algún token

## Validación Final

Antes de entregar los archivos, verificar:

- [ ] Todos los tokens tienen `$value` y `$type`
- [ ] Los tipos son solo: `color`, `number`, `string`
- [ ] Los valores de color están en formato hexadecimal válido
- [ ] Los números no tienen unidades
- [ ] La estructura es jerárquica y limpia
- [ ] Los archivos están separados por categoría

## Notas Adicionales

- Si el usuario reporta errores de importación, revisar que no haya tipos no soportados
- Preferir archivos separados para mejor organización en Figma
- Mantener nombres descriptivos pero concisos en la jerarquía

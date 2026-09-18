# Tabla de precios de Y-API — recalculada, no copiada

[English](README.md) · [简体中文](README.zh.md) · [日本語](README.ja.md) · [한국어](README.ko.md) · **Español** · [Deutsch](README.de.md) · [Português](README.pt.md)

> **Divulgación:** trabajo en Y-API, así que esta es una herramienta de primera parte leyendo un archivo de primera parte. Todo lo que imprime viene de <https://y-api.bestvirtualgoods.com/pricing.json>, que es público y no necesita clave de API: ejecuta el script y compara la salida con la fuente tú mismo.

Un script, sin dependencias, sin clave de API. Lee el archivo de precios publicado por Y-API y genera [table.md](table.md) con precios en **efectivo** — el dinero que realmente sale de tu tarjeta, y el único comparable con el precio de lista de otro proveedor.

## Por qué existe

Y-API cotiza en **crédito**, y ahora mismo $1 pagado da $20 de crédito. El crédito es lo que se descuenta del saldo; no es lo que se cobra a tu tarjeta. Una tabla que se queda en el crédito hace que cada modelo parezca 20× más caro, y una tabla que hardcodea "dividir entre 20" se vuelve silenciosamente incorrecta el día que termina la tasa promocional. Este script lee la tasa del **mismo archivo** del que lee los precios.

## Ejecútalo

```bash
node price-table.mjs > table.md                            # descarga el archivo publicado
PRICE_JSON=./pricing.json node price-table.mjs             # renderiza una copia guardada, sin red
```

Requiere Node 18+ (usa el `fetch` global). El `table.md` de este repositorio se regenera cada lunes con una GitHub Action y **solo se commitea si cambió**, así que el historial de commits *es* el registro de cambios de precio.

## Dos reglas que el script respeta

Romper cualquiera de las dos produce una tabla que contradice en silencio al sitio del que salió.

1. **El precio en efectivo se recalcula, no se copia.** El archivo publica `cash_price`, pero el script vuelve a calcular `credit_price / top_up.quota_rate` y **falla ruidosamente** si no coinciden. Ni una tasa caducada ni un número editado a mano pueden colarse en la tabla.
2. **`vendor_cheaper_on_cached_input` nunca se imprime solo.** Esa marca dice que la tarifa de entrada en caché del proveedor está por debajo de nuestro precio en efectivo: somos la opción más cara para una carga intensiva en caché. Pero algunas filas la llevan junto a `historical-price` (una tarifa antigua del proveedor), y el sitio de Y-API las excluye deliberadamente de su recuento de "perdemos en caché". **Imprimir la marca sin la advertencia hace que el lector se lleve un número equivocado.** Por eso la tabla incluye ambas columnas.

## Cómo leer la salida

- **El catálogo** — todos los modelos, crédito y efectivo, por 1M de tokens, ordenados por precio.
- **Contra el precio de lista del proveedor** — los 11 modelos con un precio de proveedor que verificamos y citamos, con el múltiplo, la marca de caché, las advertencias, la fecha de verificación y un enlace a la página del proveedor. Los 4 restantes figuran como sin precio de proveedor verificable; el archivo lo dice explícitamente en lugar de estimarlo.

`× ours` se lee como "el proveedor cobra estas veces nuestro precio en efectivo": un múltiplo grande significa que **Y-API es más barato**, no más caro.

## Alcance, con honestidad

- La tabla es tan actual como el archivo fuente. Lleva el `synced_at` del origen y la fecha de renderizado en la cabecera; si ambos son antiguos, aquí no hay novedades.
- Renderiza los precios de **un** gateway. No compara entre proveedores ni sigue el historial de precios de nadie más.
- Los precios se mueven, y la tasa de recarga es promocional sin fecha de fin anunciada. Si vas a decidir algo con esta tabla, vuelve a derivarlo de la fuente, no de una copia de `table.md`.

---

*Parte del [perfil de Y-API](https://github.com/zhuo01-l2ab).*

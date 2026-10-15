# Auditoría de catálogo · Tigo Colombia (VTEX + n8n)

Este repo es nuevo e independiente del dashboard anterior (`dianaruiz-alt/Tigo`).

Dashboard estático que lee `data/audit.json`. El archivo lo publica cada día el workflow de n8n **"Tigo · Auditoría de catálogo VTEX + IA"**.

## Puesta en marcha

1. **Crea el repo.** Créalo en GitHub como `dianaruiz-alt/tigo-auditoria-catalogo` y sube esta carpeta (`index.html`, `data/audit.json`, `README.md`).
   - El nodo de GitHub en n8n usa la operación *Edit file*, así que `data/audit.json` **debe existir antes** de la primera corrida.
   - El archivo de demo que viene incluido sirve para eso.
2. **Activa GitHub Pages.** Ve a *Settings → Pages → Deploy from branch → main / root*.
3. **En n8n:**
   1. Importa el workflow: *Workflows → Import from file → `tigo-auditoria-catalogo.n8n.json`*.
   2. Crea una credencial **Custom Auth** con este contenido y asígnala a los 5 nodos HTTP:
      ```json
      { "headers": { "X-VTEX-API-AppKey": "vtexappkey-…", "X-VTEX-API-AppToken": "…" } }
      ```
      El AppKey debe tener rol **solo lectura** (Catalog, Logistics/Inventory y Checkout de lectura).
   3. Crea la credencial de **Anthropic** (nodo *Claude*). Si prefieres Gemini, haz el cambio que explica la nota *IA* dentro del workflow.
   4. Crea la credencial de **GitHub**: un token *fine-grained* con permiso **Contents: Read and write** solo sobre este repo.
   5. Edita el nodo **Config**. Pon el `account`, y para la primera prueba usa `maxSkus = 30` y `aiMaxProductos = 5`.
4. **Ejecuta y verifica.** Lanza *Ejecutar manualmente*, revisa la salida de **Motor de reglas** y confirma el commit en el repo.
5. **Producción.** Vuelve a dejar `maxSkus = 0` (catálogo completo) y activa el workflow para que corra a diario a las 5:00 a. m. (hora Bogotá).

## Cómo se calcula el score (0–100)

| Dimensión | Puntos | Regla |
|---|---|---|
| Imágenes | 20 | ≥ 3 imágenes = 20 · 2 = 14 · 1 = 8 · 0 = 0 |
| Especificaciones | 25 | Proporción llena de las especificaciones **esperadas en su categoría** |
| Descripción | 20 | Largo de la descripción (16) + descripción corta (4). Penaliza relleno, texto igual al nombre, duplicados y HTML sucio |
| SEO | 15 | Title de 30–65 caracteres (8) + meta description de 70–160 caracteres (7) |
| Consistencia | 10 | Marca (3) + color del nombre = especificación Color (4) + nombre bien formado (3) |
| Vendibilidad | 10 | Activo (4) + stock (3) + simulación de checkout disponible y con precio > 0 (3) |

**Especificaciones esperadas.** Como no hay especificaciones obligatorias definidas, el motor las infiere del catálogo. Un campo es "esperado" en una categoría cuando al menos el **60 %** de sus SKUs lo tienen lleno (parámetro `specThreshold`). Esto aplica a categorías con 3 o más SKUs; si la categoría tiene menos, se usa el departamento.

**Semáforo.**
- **Óptimo:** score ≥ 85.
- **Mejorable:** score ≥ 60.
- **Crítico:** score < 60.

Los umbrales se configuran en el nodo **Config**.

## Sugerencias de IA

- **Qué se envía al modelo.** Solo los `aiMaxProductos` productos con más prioridad: primero los comprables, luego los que tienen más puntos por ganar en descripción y SEO.
- **Cómo se protegen los datos.** El modelo recibe únicamente los datos reales del producto y tiene instrucción de no inventar especificaciones.
- **Qué hace el flujo con las propuestas.** No escribe nada en VTEX: las propuestas se revisan en la pestaña *Optimización de contenido*, donde puedes copiarlas y marcarlas como revisadas. La marca de revisado se guarda en tu navegador.

## Archivos

- `index.html`: el dashboard. Abre `data/audit.json` o un JSON que cargues con el botón **Cargar JSON**.
- `data/audit.json`: el archivo que n8n sobrescribe en cada corrida. El que viene incluido son **datos de demostración**.

# Tarjetas del tablero de Hangar Orbital

Este archivo es la fuente que lee la skill `/tablero` para crear o actualizar el tablero **Hangar Orbital** en Trello.

**Formato:** cada tarjeta empieza con una línea que comienza con `## ` (dos almohadillas y un espacio), que es su título. La línea siguiente dice `Lista: <nombre>`. Todo lo que sigue, hasta el próximo `## `, es la descripción de la tarjeta. Dentro de una descripción, los encabezados usan `###` o más, nunca `##`.

En «Por hacer», el orden del archivo es la prioridad: la primera tarjeta se implementa primero.

## Spec 04 · Ver las misiones de una nave
Lista: Por hacer

**Historia:** HU-1.4 · Como jefe de hangar, quiero ver las misiones de cada nave, para saber qué tiene asignado antes de darle otra misión.
**Épica:** E1 · Flota y misiones. **Tickets:** T-15 a T-17.

### 0. Decisiones de esta spec
1. **Se muestran todas las misiones de la nave.** No existe estado de misión (Spec 01, decisión 2), así que «pendiente» equivale a «asignada» hasta que exista HU-1.5. `[SUPUESTO]`
2. **Orden:** por fecha de despegue, de la más próxima a la más lejana. Si dos misiones despegan el mismo día, va primero la que se asignó antes (`createdAt`).
3. **Carga bajo demanda:** las misiones de una nave se cargan al pulsar «Ver misiones», no al abrir la flota.
4. **Solo lectura:** esta spec no cambia el contador, la aptitud ni la base de datos. No necesita migración: la política de lectura de `misiones` ya existe.

### 1. Funciones puras (`src/dominio/mision.ts`)
- `ordenarMisiones(misiones: Mision[]): Mision[]`: devuelve un arreglo nuevo ordenado según la decisión 2. NO DEBE modificar el arreglo recibido.
- `describirFechaDespegue(fecha: string, hoy: string): string`: ambas en `YYYY-MM-DD`. Devuelve «Hoy» si coinciden, «Mañana» si `fecha` es el día siguiente a `hoy`, «Despegó el DD/MM/AAAA» si `fecha` es anterior a `hoy`, y «DD/MM/AAAA» en cualquier otro caso.
- `describirDuracion(dias: number): string`: «1 día» para 1 y «N días» para cualquier otro entero.

### 2. Servicio (`src/services/misionesService.ts`)
- `listarPorNave(naveId: string): Promise<Mision[]>`: lee las misiones de la nave con las columnas de `COLUMNAS_MISION` y las devuelve con `aMision`, sin ordenar. Si Supabase devuelve un error, lanza un `Error` con su mensaje.

### 3. Componente (`src/components/MisionesNave.tsx`)
- Se muestra como **último bloque** de la Tarjeta de nave, después de los motivos. Antes de escribir código, se añade este bloque 7 a la sección 6.7 de `DESIGN.md`.
- Un Botón secundario (DESIGN.md §6.3) con el texto «Ver misiones», que cambia a «Ocultar misiones» cuando la lista está abierta.
- Al abrir: «Cargando misiones…» mientras carga; «No se pudieron cargar las misiones.» como aviso de error (DESIGN.md §6.8) si falla; «Esta nave no tiene misiones.» si no hay ninguna.
- Cada misión es un `li` con el destino, la fecha de `describirFechaDespegue` en `--fuente-cifras` y la duración de `describirDuracion`.
- Solo usa tokens de `DESIGN.md`. La fecha de hoy se obtiene con `fechaLocalHoy()` de `src/lib/fecha.ts`.

### 4. Criterios de aceptación
- **HU-1.4-CA1:** con una nave que tiene dos misiones con despegue el 20/12/2026 y el 10/12/2026, al pulsar «Ver misiones» la app DEBE listar las dos, primero la del 10/12/2026.
- **HU-1.4-CA2:** con dos misiones que despegan el mismo día, la app DEBE mostrar primero la que se asignó antes.
- **HU-1.4-CA3:** con una nave sin misiones, la app DEBE mostrar «Esta nave no tiene misiones.».
- **HU-1.4-CA4:** una misión que despega hoy DEBE mostrar «Hoy», y una que despega mañana, «Mañana».
- **HU-1.4-CA5:** una misión con fecha de despegue anterior a hoy DEBE mostrar «Despegó el DD/MM/AAAA».
- **HU-1.4-CA6:** una misión de 1 día DEBE mostrar «1 día», y una de 5 días, «5 días».
- **HU-1.4-CA7:** si falla la carga, la app DEBE mostrar «No se pudieron cargar las misiones.», y el resto de la tarjeta DEBE seguir visible.
- **HU-1.4-CA8:** ver las misiones NO DEBE cambiar el contador ni la aptitud de la nave.

### 5. Verificación
| Criterio | Cómo |
| --- | --- |
| CA1, CA2 | Vitest: `ordenarMisiones` (incluye que no modifica el arreglo original) |
| CA4, CA5 | Vitest: `describirFechaDespegue` (hoy, mañana, ayer, fecha futura, cambio de mes y de año) |
| CA6 | Vitest: `describirDuracion` |
| CA3, CA7, CA8 | Manual en el navegador |

### 6. Tickets y archivos
| Ticket | Crea | Modifica |
| --- | --- | --- |
| T-15 · Reglas de presentación de misiones | — | `src/dominio/mision.ts`, `src/dominio/mision.test.ts` |
| T-16 · Listar y mostrar las misiones | `src/components/MisionesNave.tsx`, `src/styles/misiones.css` | `src/services/misionesService.ts`, `src/components/TarjetaNave.tsx`, `DESIGN.md` (§6.7, bloque 7) |
| T-17 · Verificar HU-1.4 | — | Esta spec (resultados de la prueba manual) |

### Pendientes
(La implementación anota aquí lo que encuentre fuera de alcance.)

## Spec 05 · Registrar una revisión
Lista: Por hacer

**Historia:** HU-2.1 · Como jefe de hangar, quiero registrar que una nave pasó revisión, para que pueda volver a recibir misiones.
**Épica:** E2 · Mantenimiento. **Regla:** R5 (una revisión pone el contador de misiones a cero). **Tickets:** T-18 y T-19.

### 0. Decisiones de esta spec
1. **Registrar una revisión actualiza `ultima_revision` a la fecha y hora actuales.** El contador cuenta las misiones creadas después de esa fecha (Spec 01, decisión 1), así que vuelve a 0 sin guardar ningún contador.
2. **Una nave en reparación no puede registrar revisión:** salir del dique seco ya cuenta como revisión (R7), que se implementará en otra spec.
3. **La fecha la pone la base de datos** con `now()`, no el navegador, para que no dependa del reloj del jefe.

### 1. Base de datos
Archivo nuevo: `supabase/migraciones/002_registrar_revision.sql`.
- Una función `registrar_revision(p_nave_id uuid)` con `security invoker` que actualiza `ultima_revision = now()` solo si la nave existe y `en_reparacion` es falso, y devuelve la fila actualizada.
- Permiso para `anon`: `update` únicamente sobre la columna `ultima_revision` de `naves`, con su política RLS de actualización. Ninguna otra columna se puede actualizar.

### 2. Funciones puras (`src/dominio/revision.ts`)
- `puedeRegistrarRevision(nave: Pick<Nave, 'enReparacion'>): { puede: true } | { puede: false; motivo: string }`. Si la nave está en reparación, el motivo es «La nave está en reparación: la revisión se registra al salir del dique seco.».

### 3. Servicio (`src/services/navesService.ts`)
- `registrarRevision(naveId: string): Promise<Nave>`: llama a la función `registrar_revision` con `rpc` y devuelve la nave con `aNave`. Lanza un `Error` si Supabase falla o si no devuelve ninguna fila.

### 4. Componente
- En la Tarjeta de nave, un Botón secundario «Registrar revisión». Si `puedeRegistrarRevision` dice que no, el botón aparece deshabilitado y el motivo se muestra debajo.
- Al terminar: aviso de éxito «Revisión registrada.» y la tarjeta se actualiza (contador y aptitud). Si falla: aviso de error «No se pudo registrar la revisión.».

### 5. Criterios de aceptación
- **HU-2.1-CA1:** con una nave al 60 % y 3 misiones desde la última revisión, al registrar una revisión la nave DEBE pasar a «0 de 3» y a «Apta».
- **HU-2.1-CA2:** con una nave al 10 % y 3 misiones, al registrar una revisión DEBE pasar a «0 de 3» y DEBE seguir «No apta» por combustible insuficiente.
- **HU-2.1-CA3:** con una nave en reparación, el botón DEBE estar deshabilitado y DEBE mostrarse el motivo.
- **HU-2.1-CA4:** después de registrar la revisión, la app DEBE mostrar «Revisión registrada.».
- **HU-2.1-CA5:** una misión asignada después de la revisión DEBE contar como «1 de 3».

### 6. Tickets y archivos
| Ticket | Crea | Modifica |
| --- | --- | --- |
| T-18 · Regla y servicio de revisión | `supabase/migraciones/002_registrar_revision.sql`, `src/dominio/revision.ts`, `src/dominio/revision.test.ts` | `src/services/navesService.ts` |
| T-19 · Botón y verificación | — | `src/components/TarjetaNave.tsx`, `src/components/ListaNaves.tsx`, esta spec (resultados) |

### Pendientes
(La implementación anota aquí lo que encuentre fuera de alcance.)

## Spec 06 · Corregir el combustible
Lista: Por hacer

**Historia:** HU-2.2 · Como jefe de hangar, quiero corregir el combustible de una nave, para que su aptitud refleje el nivel real.
**Épica:** E2 · Mantenimiento. **Regla:** R4 (el combustible se puede corregir en cualquier momento). **Tickets:** T-20 y T-21.

### 0. Decisiones de esta spec
1. **Mismas reglas que al registrar:** entero entre 0 y 100, con los mismos mensajes de `validarNave` (HU-1.1-CA7 y CA8).
2. **Un texto con solo espacios cuenta como vacío.**
3. **Se puede corregir aunque la nave esté en reparación.** R4 dice «en cualquier momento».

### 1. Base de datos
Archivo nuevo: `supabase/migraciones/003_corregir_combustible.sql`. Permiso para `anon`: `update` únicamente sobre la columna `combustible` de `naves`, con su política RLS. El `check` de la columna (0 a 100) ya existe.

### 2. Funciones puras (`src/dominio/nave.ts`)
- `validarCombustible(texto: string): { ok: true; valor: number } | { ok: false; error: string }`. Vacío o solo espacios: «Indica el combustible de la nave.». No numérico, decimal, negativo o mayor que 100: «El combustible debe ser un número entero entre 0 y 100.». `validarNave` pasa a usar los mismos textos, sin duplicarlos.

### 3. Servicio (`src/services/navesService.ts`)
- `corregirCombustible(naveId: string, combustible: number): Promise<Nave>`: actualiza solo `combustible` y devuelve la nave con `aNave`.

### 4. Componente
- En la Tarjeta de nave, un Campo de formulario «Corregir combustible» con un Botón secundario «Guardar». Los errores se muestran bajo el campo (DESIGN.md §6.2). Al guardar, la barra de combustible y la aptitud se actualizan.

### 5. Criterios de aceptación
- **HU-2.2-CA1:** al corregir de 24 % a 25 % una nave operativa con menos de 3 misiones, DEBE pasar de «No apta» a «Apta».
- **HU-2.2-CA2:** al corregir de 60 % a 10 %, DEBE pasar a «No apta» por combustible insuficiente.
- **HU-2.2-CA3:** con el campo vacío o con solo espacios, la app NO DEBE guardar y DEBE mostrar «Indica el combustible de la nave.».
- **HU-2.2-CA4:** con -1, 101, 24,5 o «abc», la app NO DEBE guardar y DEBE mostrar el mensaje de rango.
- **HU-2.2-CA5:** con 0 y con 100, la app DEBE guardar.
- **HU-2.2-CA6:** una nave en reparación DEBE poder corregir su combustible.

### 6. Tickets y archivos
| Ticket | Crea | Modifica |
| --- | --- | --- |
| T-20 · Regla y servicio de combustible | `supabase/migraciones/003_corregir_combustible.sql` | `src/dominio/nave.ts`, `src/dominio/nave.test.ts`, `src/services/navesService.ts` |
| T-21 · Campo y verificación | — | `src/components/TarjetaNave.tsx`, `src/components/ListaNaves.tsx`, esta spec (resultados) |

### Pendientes
(La implementación anota aquí lo que encuentre fuera de alcance.)

## Spec 07 · Completar una misión
Lista: Por hacer

**Historia:** HU-1.5 · Como jefe de hangar, quiero marcar una misión pendiente como completada, para dejar constancia de que la nave volvió y anotar el combustible que le queda.
**Épica:** E1 · Flota y misiones. **Reglas:** R3 (completar no cambia el contador) y R4 (el combustible restante es obligatorio y reemplaza el de la nave). **Tickets:** T-22 a T-24.
**Criterios:** HU-1.5-CA1 a CA7, en `docs/backlog/E1-flota-y-misiones.md`. **Wireframes aprobados:** `docs/specs/07-completar-mision/wireframes/`.

### 0. Decisiones de esta spec
1. **Una misión está «pendiente» o «completada».** Se guarda en una columna nueva, `estado`, de `misiones`. Las misiones que ya existen pasan a «pendiente». Se reemplaza la decisión 1 de la Spec 04, según la cual «pendiente» equivalía a «asignada».
2. **Al completar, el combustible restante es obligatorio y reemplaza el de la nave (R4).** Se valida con `validarCombustible` y sus mensajes (Spec 06). Se acepta un valor mayor que el que tenía la nave, porque pudo repostar fuera.
3. **Completar no cambia el contador (R3).** El contador cuenta las misiones creadas después de la última revisión, sin mirar su estado (Spec 01, decisión 1). Ninguna consulta del contador se toca.
4. **Solo se completa una misión pendiente con despegue hoy o anterior.** La de mañana o después no muestra «Completar».
5. **Se completa desde la lista de misiones de la nave (Spec 04), una misión a la vez.** La misión completada sigue en la lista, marcada «Completada» y sin ninguna acción.
6. **Orden de la lista:** primero las pendientes y después las completadas. Cada grupo mantiene el orden de la decisión 2 de la Spec 04.
7. **Una sola operación:** marcar la misión y reemplazar el combustible ocurren en la misma función de la base de datos. Si una de las dos falla, no cambia ninguna.
8. **`[SUPUESTO]` Misión ya completada por otro jefe:** si la misión ya no está pendiente al confirmar, la función no cambia nada y la app muestra el error de conexión (CA6). Falta que el cliente lo confirme (caso borde de HU-1.5).
9. **`[SUPUESTO]` Nave en reparación:** puede completar una misión, porque R4 permite cambiar el combustible en cualquier momento y las decisiones del cliente no lo impiden. Falta que el cliente lo confirme.
10. **`[SUPUESTO]` Error al guardar:** aviso de error «No se pudo completar la misión.», sin aviso de éxito, igual que en la Spec 06.

### 1. Pantallas
Los wireframes de `docs/specs/07-completar-mision/wireframes/` deciden la estructura de esta spec (AGENTS.md, Reglas).

| Estado | Wireframes | Criterios |
| --- | --- | --- |
| `lista` | `lista-movil.svg`, `lista-escritorio.svg` | HU-1.5-CA1, CA7 |
| `formulario` | `pedir-combustible-movil.svg`, `pedir-combustible-escritorio.svg` | HU-1.5-CA2, CA7 |
| `error-validacion` | `error-validacion-movil.svg` | HU-1.5-CA3, CA7 |
| `guardando` | `guardando-movil.svg` | HU-1.5-CA4 |
| `error-conexion` | `error-conexion-movil.svg` | HU-1.5-CA6 |
| `exito` | `completada-movil.svg` | HU-1.5-CA5, CA7 |

**Es normativo:**
- La estructura: qué bloques hay y qué contiene cada uno.
- El orden de los bloques, de las misiones y de los grupos «Pendientes» y «Completadas».
- Los textos: etiquetas, botones, estados y mensajes.
- Las 10 decisiones de `wireframes/README.md`. Entre ellas:
  - el campo se abre dentro de la misión, en lugar de «Completar»;
  - «Confirmar» es primario y «Cancelar» es secundario;
  - en «Guardando» se deshabilitan los dos botones;
  - el aviso de error va debajo de los botones;
  - no hay aviso de éxito.

**No es normativo:** los colores, la tipografía, los tamaños de letra y los grises de los wireframes. Salen de los tokens y los componentes de DESIGN.md.

**Antes de escribir código, se actualiza DESIGN.md:**
- **§6.3 Botón:** se añade el Botón primario «Confirmar» dentro de una misión, como excepción a «primario solo en paneles», y «Completar» y «Cancelar» como secundarios.
- **§6.4 Insignia de estado:** se añade una variante neutra para el estado de una misión («Pendiente», «Completada»), con texto obligatorio y sin color de aptitud.
- **§6.7 Tarjeta de nave, bloque 9 «Misiones»:**
  - los subtítulos «Pendientes» y «Completadas»;
  - el estado de cada misión;
  - la fecha y la duración en dos líneas;
  - el botón «Completar»;
  - el formulario con el campo «Combustible restante (%)» (§6.2, texto con `inputMode="numeric"`), «Confirmar» y «Cancelar».
- **§6.8 Avisos:** el aviso de error de completar va debajo de «Confirmar» y «Cancelar», dentro de la misión.

### 2. Base de datos
Archivo nuevo: `supabase/migraciones/004_completar_mision.sql`.

- **Estado de la misión:** columna `estado text not null default 'pendiente'`, con un `check (estado in ('pendiente', 'completada'))`.
- **Función `completar_mision(p_mision_id uuid, p_combustible integer) returns setof naves`:**
  - Usa `security definer` y `set search_path = public`.
  - En **una sola sentencia**, un `with` hace dos cosas:
    - pasa la misión a `'completada'` solo si `estado = 'pendiente'` y `fecha_despegue <= current_date`;
    - con el `nave_id` que devuelve, actualiza `naves.combustible = p_combustible`.
  - Devuelve la nave actualizada.
  - No devuelve ninguna fila si la misión no existe, ya está completada o todavía no ha despegado.
  - Si `p_combustible` está fuera de 0–100, el `check` de la columna hace fallar la sentencia entera y no cambia nada.
- **Permisos: no se abre ninguno nuevo sobre las tablas.**
  - Las Specs 05 y 06 muestran que las políticas de `update` se suman por fila y que un permiso por columna acabó abriendo más de lo previsto. Por eso **no** se concede `update` sobre `misiones` a `anon` ni se crea ninguna política de `update`. Solo la función cambia el estado, y lo hace con `security definer`.
  - `revoke execute on function completar_mision(uuid, integer) from public, anon, authenticated;` y después `grant execute ... to anon;`. Supabase concede `execute` a `anon` por defecto; por eso no basta con revocar a `public`.
  - **Inserción:** hoy `anon` puede insertar cualquier columna de `misiones`, así que podría crear una misión ya «completada». Se cambia a `revoke insert on misiones from anon;` y `grant insert (nave_id, destino, fecha_despegue, duracion_dias) on misiones to anon;`. Hay que comprobar que `misionesService.crear` solo inserta esas cuatro columnas.
- **Fuera de alcance:** el pendiente de las Specs 05 y 06 sigue abierto. Sin inicio de sesión, cualquiera con la clave pública puede llamar a la función. Cuando haya usuarios, `execute` pasa a `authenticated`.

### 3. Tipos (`src/types/mision.ts`)
- `export type EstadoMision = 'pendiente' | 'completada';`
- `Mision` gana `estado: EstadoMision`.

### 4. Funciones puras (`src/dominio/mision.ts`)
- `agruparMisiones(misiones: Mision[]): { pendientes: Mision[]; completadas: Mision[] }`: separa por estado y ordena cada grupo con `ordenarMisiones`. NO DEBE modificar el arreglo recibido.
- `puedeCompletarMision(mision: Pick<Mision, 'estado' | 'fechaDespegue'>, hoy: string): boolean`: verdadero solo si la misión está pendiente y `fechaDespegue <= hoy`. Las dos fechas van en `YYYY-MM-DD`.
- `describirEstadoMision(estado: EstadoMision): string`: devuelve «Pendiente» o «Completada».
- Se reutiliza `validarCombustible` de `src/dominio/nave.ts`, sin cambios.

### 5. Servicio (`src/services/misionesService.ts`)
- `FilaMision`, `COLUMNAS_MISION` y `aMision` incluyen `estado`.
- `completar(misionId: string, combustible: number): Promise<Nave>`:
  - llama a `completar_mision` con `rpc` y devuelve la nave con `aNave`;
  - comprueba el resultado como `unknown` antes de usarlo, como `registrarRevision`;
  - lanza un `Error` si Supabase falla o si no devuelve ninguna fila.

### 6. Componentes
- **`src/components/ListaNaves.tsx`:** nuevo `completarMision(naveId, misionId, combustible)`.
  - Llama a `misionesService.completar` y reemplaza la nave en la flota con la que devuelve.
  - Así la barra, la insignia y los motivos se actualizan sin recargar.
  - Si falla, relanza el error para que lo muestre la misión.
- **`src/components/TarjetaNave.tsx`:** recibe `onCompletarMision` y lo pasa a `MisionesNave`.
- **`src/components/MisionesNave.tsx`:**
  - Muestra los grupos de `agruparMisiones` con los subtítulos «Pendientes» y «Completadas». Un grupo vacío no muestra su subtítulo.
  - Cada misión muestra el estado de `describirEstadoMision`.
  - Muestra «Completar» solo si `puedeCompletarMision(mision, fechaLocalHoy())`.
  - Guarda qué misión tiene el formulario abierto. Abrir otra cierra la anterior (una a la vez).
  - Al terminar con éxito, marca la misión como completada en su estado local, así pasa al grupo «Completadas» sin recargar la lista.
- **`src/components/CompletarMision.tsx` (nuevo):** el formulario dentro de la misión.
  - **Campo:** «Combustible restante (%)», de tipo texto con `inputMode="numeric"`, como en la Spec 06.
  - **Validación:** `validarCombustible`. El error va debajo del campo, con `aria-invalid` y `aria-describedby`, y conserva lo escrito.
  - **Botones:** «Confirmar» primario y «Cancelar» secundario. «Cancelar» cierra el formulario sin guardar.
  - **Guardando:** el campo y los dos botones se deshabilitan, sin cambiar de texto ni de ancho. Un segundo clic no envía nada.
  - **Error:** muestra el aviso «No se pudo completar la misión.» con `role="alert"` debajo de los botones. Conserva el valor escrito y vuelve a habilitar los botones.
- **`src/styles/misiones.css`:** estilos de los grupos, del estado de la misión y del formulario. Solo usa tokens de DESIGN.md.

### 7. Criterios de aceptación
Los de HU-1.5 en `docs/backlog/E1-flota-y-misiones.md`, sin cambios:
- **CA1:** lista con misiones pendientes y completadas.
- **CA2:** pedir el combustible restante.
- **CA3:** error de validación.
- **CA4:** guardando.
- **CA5:** misión completada (R3, R4).
- **CA6:** error de conexión.
- **CA7:** uso en el celular a 390 px.

### 8. Verificación
**Vitest:**

| Criterio | Qué se prueba |
| --- | --- |
| CA1 | `agruparMisiones`: pendientes antes que completadas, el orden dentro de cada grupo y que no modifica el arreglo original. `puedeCompletarMision`: pendiente de ayer y de hoy (sí), pendiente de mañana (no), completada de ayer (no). `describirEstadoMision`. |
| CA3 | `validarCombustible` ya tiene tests (Spec 06). Se añaden los casos de CA3: vacío, solo espacios, «-5», «101», «24,5» y «abc». |

**Base de datos**, en el editor SQL de Supabase. Se anota el resultado en «Resultados»:
- Completar una misión pendiente que ya despegó cambia su estado y el combustible de la nave, y no cambia el contador.
- Completarla otra vez no devuelve filas y no cambia nada (decisión 8).
- Una misión de mañana no se completa.
- Un combustible de 101 no cambia ni la misión ni la nave.
- Como `anon`, un `update` directo de `misiones.estado` se rechaza, y un `insert` con `estado = 'completada'` también.

**Datos de prueba**, cargados a mano en Supabase como en T-07, porque la app no permite fechas pasadas:
- **Aurora:** carguera, 60 %, operativa. Su `ultima_revision` es posterior a Europa y anterior a Luna y Marte, así que muestra «2 de 3». Sus misiones:
  - Europa: completada, despegó hace 5 días.
  - Luna: pendiente, despegó hace 3 días, 5 días de duración.
  - Marte: pendiente, despega mañana, 3 días de duración.
- **Orión:** transporte, 80 %, 1 misión.
- **Vega:** exploradora, 18 %, sin misiones.

Aurora es la primera de la flota por nombre, así que «clic:Ver misiones» abre sus misiones.

**Capturas.** Con `npm run dev` corriendo, en este orden:
1. `npm run captura -- 07-completar-mision lista "clic:Ver misiones" "esperar:Pendientes"`
2. `npm run captura -- 07-completar-mision formulario "clic:Ver misiones" "esperar:Pendientes" "clic:Completar"`
3. `npm run captura -- 07-completar-mision error-validacion --anchos 390 "clic:Ver misiones" "esperar:Pendientes" "clic:Completar" "escribir:Combustible restante (%)=101" "clic:Confirmar" "esperar:El combustible debe ser un número entero entre 0 y 100."`
4. `npm run captura -- 07-completar-mision guardando --anchos 390 "clic:Ver misiones" "esperar:Pendientes" "clic:Completar" "escribir:Combustible restante (%)=10" "lento" "clic:Confirmar"`
5. `npm run captura -- 07-completar-mision error-conexion --anchos 390 "clic:Ver misiones" "esperar:Pendientes" "clic:Completar" "escribir:Combustible restante (%)=10" "sin-red" "clic:Confirmar" "esperar:No se pudo completar la misión."`
6. `npm run captura -- 07-completar-mision exito --anchos 390 "clic:Ver misiones" "esperar:Pendientes" "clic:Completar" "escribir:Combustible restante (%)=10" "clic:Confirmar" "esperar:10 %"`

Notas sobre las capturas:
- **`exito` va al final** porque completa Luna de verdad. Para repetir las capturas hay que volver a cargar los datos de prueba.
- **`guardando` no debería completar Luna**, porque la captura cierra la página antes de que pasen los 5 s de «lento». Si el paso 5 falla porque Luna ya no tiene «Completar», hay que volver a cargar los datos.
- **Comparación:** cada captura se compara con su wireframe en estructura, orden y textos, y el resultado se anota en «Resultados».

**En el navegador**, lo que una captura no muestra:
- **CA2:** «Cancelar» cierra el formulario y vuelve a mostrar «Completar».
- **CA4:** un doble clic en «Confirmar» completa la misión una sola vez.
- **CA5:** después de recargar la página, Luna sigue «Completada», Aurora muestra 10 % y el contador sigue en «2 de 3».
- **CA7:** a 390 px no hay scroll horizontal, los botones miden al menos 40 × 40 px y el campo abre el teclado numérico en un celular o en el modo responsive con táctil.

### 9. Tickets y archivos
| Ticket | Crea | Modifica |
| --- | --- | --- |
| T-22 · Estado de la misión, reglas y servicio | `supabase/migraciones/004_completar_mision.sql` | `src/types/mision.ts`, `src/dominio/mision.ts`, `src/dominio/mision.test.ts`, `src/dominio/nave.test.ts`, `src/services/misionesService.ts` |
| T-23 · Completar desde la lista de misiones | `src/components/CompletarMision.tsx` | `DESIGN.md` (§6.3, §6.4, §6.7 bloque 9, §6.8), `src/components/MisionesNave.tsx`, `src/components/TarjetaNave.tsx`, `src/components/ListaNaves.tsx`, `src/styles/misiones.css` |
| T-24 · Verificar HU-1.5 | `docs/specs/07-completar-mision/capturas/` (las capturas de §8) | Esta spec (resultados de la base de datos, de las capturas y del navegador) |

### Pendientes
(La implementación anota aquí lo que encuentre fuera de alcance.)

## HU-2.3 · Poner una nave en dique seco y sacarla
Lista: Backlog

Como jefe de hangar, quiero poner una nave en dique seco y sacarla de él, para que no reciba misiones mientras se repara.

- **Reglas:** R6 (al entrar pierde sus misiones pendientes y no recibe nuevas) y R7 (salir cuenta como revisión).
- **Falta antes de la spec:** qué significa «pierde sus misiones pendientes» si las misiones no tienen estado.

## Decisión · ¿«4 de 3» es posible?
Lista: Backlog

La Spec 02 dejó como `[SUPUESTO]` que el contador puede mostrar más misiones que el máximo, por ejemplo «4 de 3», si llegan datos antiguos. Hay que confirmarlo con el cliente antes de cerrar E1.

## HU-1.9 · Consultar el historial de misiones
Lista: Backlog

Como jefe de hangar, quiero ver las misiones completadas de una nave, para revisar su actividad pasada.

- **Falta antes de la spec:** el PRD tiene abierto si se conserva el historial. Depende de HU-1.5.

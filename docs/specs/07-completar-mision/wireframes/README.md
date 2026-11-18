# Wireframes · HU-1.5 · Completar una misión

Wireframes de baja fidelidad en escala de grises, hechos a partir de los criterios de HU-1.5 en [docs/backlog/E1-flota-y-misiones.md](../../../backlog/E1-flota-y-misiones.md). No usan los colores ni las fuentes de `DESIGN.md`; solo sus componentes y sus textos.

Todos los estados usan la misma nave de ejemplo: Aurora, carguera, 60 % de combustible, «2 de 3» misiones y operativa. Sus misiones son:
- Luna: pendiente, despegó el 05/12/2026.
- Marte: pendiente, despega mañana.
- Europa: completada.

| Estado | Archivos | Criterios |
| --- | --- | --- |
| Lista con misiones pendientes y completadas | [lista-movil.svg](lista-movil.svg), [lista-escritorio.svg](lista-escritorio.svg) | HU-1.5-CA1, HU-1.5-CA7 |
| Pedir el combustible restante | [pedir-combustible-movil.svg](pedir-combustible-movil.svg), [pedir-combustible-escritorio.svg](pedir-combustible-escritorio.svg) | HU-1.5-CA2, HU-1.5-CA7 |
| Error de validación | [error-validacion-movil.svg](error-validacion-movil.svg) | HU-1.5-CA3, HU-1.5-CA7 |
| Guardando | [guardando-movil.svg](guardando-movil.svg) | HU-1.5-CA4 |
| Misión completada | [completada-movil.svg](completada-movil.svg) | HU-1.5-CA5, HU-1.5-CA7 |
| Error de conexión | [error-conexion-movil.svg](error-conexion-movil.svg) | HU-1.5-CA6 |

## Decisiones
Esto lo decide el wireframe; los criterios no lo dicen.

1. **Dónde se pide el combustible:** dentro de la misma misión, debajo de su destino, fecha y duración. El campo ocupa el lugar del botón «Completar». No hay ventana emergente ni panel aparte.
2. **Botón primario:** «Confirmar» es primario y «Cancelar» es secundario. Esto contradice la tabla de DESIGN.md §6.3, donde el primario solo se usa en las acciones principales de los paneles. Si se aprueba, la spec tiene que ampliar §6.3.
3. **Grupos de la lista:** las pendientes y las completadas se separan con los subtítulos «Pendientes» y «Completadas». Un grupo sin misiones no muestra su subtítulo.
4. **Estado de cada misión:** se escribe como una etiqueta con borde, a la derecha del destino, con la palabra «Pendiente» o «Completada».
5. **Fecha y duración en dos líneas:** en una tarjeta de escritorio de unos 260 px, «Despegó el 05/12/2026 · 5 días» no cabe en una sola línea.
6. **Guardando:** «Confirmar» y «Cancelar» se deshabilitan los dos, y el campo queda en gris con el valor escrito. El criterio solo habla de «Confirmar».
7. **Error de validación:** el campo tiene un borde de 2 px y el mensaje aparece debajo. El ejemplo es «101», que muestra «El combustible debe ser un número entero entre 0 y 100.». El campo vacío usa la misma disposición con «Indica el combustible de la nave.».
8. **Error de conexión:** el aviso aparece dentro de la misión, debajo de «Confirmar» y «Cancelar», siguiendo DESIGN.md §6.8 («debajo del botón que lo produjo»).
9. **Sin aviso de éxito:** al completar, el cambio se ve en la propia tarjeta:
   - la misión pasa al grupo «Completadas»;
   - la barra baja a 10 %;
   - aparece «NO APTA» con su motivo.

   Es el mismo criterio que la spec 06 aplica al corregir el combustible.
10. **Escritorio:** solo la tarjeta de Aurora tiene las misiones abiertas. Las demás naves muestran «Ver misiones», así que la grilla solo crece en una columna.

## Cobertura de los criterios
Cada criterio aparece en al menos un wireframe. Estas partes no se pueden dibujar y se comprueban en el navegador:
- **HU-1.5-CA2:** el resultado de «Cancelar» es volver al estado «Lista»; no tiene un dibujo propio.
- **HU-1.5-CA4:** que pulsar de nuevo no envíe una segunda vez es un comportamiento, no algo que se vea.
- **HU-1.5-CA7:** que el campo abra el teclado numérico. Los 40 px de alto de los botones sí están dibujados.

## Puntos para revisar
- **Dos campos de combustible en la tarjeta:** mientras se completa una misión, la tarjeta muestra «Corregir combustible» (spec 06) y «Combustible restante (%)» a la vez. Hay que confirmar que el jefe no los confunde.

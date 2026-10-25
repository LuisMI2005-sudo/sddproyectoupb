# Tareas: MVP de habits-cli

Fuentes: `spec.md`, `plan.md` y la constitución del proyecto.
Este documento describe trabajo futuro; no autoriza ni ejecuta implementación.

Cada tarea representa entre 15 y 30 minutos de trabajo activo, sin contar esperas
por respuestas. El orden respeta las dependencias indicadas. Una tarea solo puede
marcarse cuando se cumpla su línea «Hecho cuando:»; las tareas de implementación
incluyen sus pruebas pertinentes y no se consideran terminadas con pruebas fallidas.

Preparación de las próximas tareas con pruebas: instalar primero la dependencia
de desarrollo con `pip install pytest` en el entorno de trabajo y después ejecutar
`pytest -q`. Si se usa `python -m pip install pytest`, debe corresponder al mismo
intérprete que ejecuta las pruebas. La instalación no añade dependencias de
ejecución a la aplicación ni implica completar T08.

## 1. Aclaraciones y coherencia documental

Las aclaraciones proceden de la sección 5 del plan. No se inventarán respuestas
ni se considerarán aprobadas las propuestas por estar incluidas en el plan.
Estas tareas se ejecutarán cuando se solicite resolver las dudas; este documento
no cambia la especificación ni el plan.

- [ ] **T01 — Concretar las reglas pendientes de nombres y ordenación (25 min).**
  RF: RF-2, RF-3, RF-8. Dependencias: ninguna.
  Recoger las decisiones sobre caracteres de control, longitud, equivalencia Unicode y orden de tildes y ñ.
  Hecho cuando: cada punto tenga una respuesta explícita del usuario y al menos un ejemplo con resultado esperado; ningún punto quede resuelto por suposición.

- [x] **T02 — Concretar el tratamiento de cambios del reloj (20 min).**
  RF: RF-6, RF-7. Dependencias: ninguna.
  Recoger el comportamiento aprobado cuando registros existentes quedan en el futuro por cambios de fecha o zona horaria.
  Hecho cuando: un ejemplo de retroceso de fecha y otro de cambio de zona horaria tengan resultados esperados aprobados para consulta y registro.
  Evidencia: el usuario aprobó ambos escenarios; están registrados en RF-6, RF-7 y los casos límite de `spec.md`, y la decisión resuelta consta en la sección 5 de `plan.md`. Las marcas futuras se conservan, no cuentan hasta que corresponda y no impiden marcar hoy o detectar su repetición. T02 es documental; no acredita implementación ni pruebas automatizadas.

- [ ] **T03 — Concretar la desaparición de datos anteriores (20 min).**
  RF: RF-9, RF-10. Dependencias: ninguna.
  Distinguir el primer uso del escenario en que un usuario pierde sus datos guardados.
  Hecho cuando: ambos escenarios tengan un resultado observable aprobado, incluido si permiten crear nuevos hábitos.

- [ ] **T04 — Concretar el alcance del uso simultáneo (20 min).**
  RF: RF-9, RF-10. Dependencias: ninguna.
  Obtener la decisión sobre dos procesos que intentan modificar los mismos datos, sin equiparar reemplazo atómico con protección de cambios concurrentes.
  Hecho cuando: exista un resultado esperado aprobado para dos creaciones y dos marcas simultáneas, o una exclusión explícitamente aprobada de ese escenario.

- [ ] **T05 — Concretar las garantías ante interrupciones (20 min).**
  RF: RF-9, RF-10. Dependencias: ninguna.
  Recoger el resultado esperado tras interrumpir el guardado y al volver a ejecutar la aplicación, distinguiendo interrupción de proceso y corte de energía.
  Hecho cuando: se documenten las garantías aprobadas y sus límites, sin prometer durabilidad universal.

- [ ] **T06 — Concretar la revisión de mensajes (15 min).**
  RF: RF-1, RF-11; también RNF-1. Dependencias: ninguna.
  Recoger el criterio de comprensibilidad aplicable a ayuda, confirmaciones y errores.
  Hecho cuando: exista un criterio aprobado que permita decidir si un mensaje cumple o no, acompañado de ejemplos.

- [ ] **T07 — Reconciliar especificación y plan (30 min).**
  RF: RF-1 a RF-11. Dependencias: T01, T02, T03, T04, T05, T06.
  Incorporar las decisiones aprobadas y reconciliar el esquema, las validaciones y el contrato observable propuestos en el plan con la especificación, previa autorización para editar los documentos afectados.
  Hecho cuando: no haya decisiones contradictorias ni aclaraciones bloqueantes; cada comportamiento nuevo tenga su criterio de aceptación y las decisiones que requieran trabajo adicional tengan tareas de hasta 30 minutos insertadas antes de sus consumidoras.

No se estima como una sola tarea la implementación de una política todavía
desconocida. Si T01–T06 amplían el alcance técnico, T07 debe dejar el desglose
adicional concreto antes de comenzar el trabajo afectado. Las tareas siguientes
desglosan el alcance técnico ya descrito en el plan.

## 2. Base del proyecto y lógica de negocio

- [ ] **T08 — Preparar el paquete y el entorno de pruebas (20 min).**
  RF: RF-1, RF-11, soporte transversal. Dependencias: T07.
  Preparar los módulos previstos, las carpetas de pruebas y la disponibilidad de pytest, sin dependencias de ejecución adicionales.
  Hecho cuando: el paquete pueda importarse sin salida ni acceso a datos, y pytest esté disponible para ejecutar las pruebas posteriores.

- [ ] **T09 — Implementar identificación y validación de nombres (25 min).**
  RF: RF-2. Dependencias: T08.
  Separar nombre de presentación y comparación; aplicar las reglas de nombres aprobadas con pruebas unitarias.
  Hecho cuando: «Leer» y « leer » identifiquen el mismo hábito, se conserven las mayúsculas de presentación y se rechacen nombres vacíos sin mutar las entradas; los casos adicionales de T01 también pasen.

- [ ] **T10 — Implementar la creación en el núcleo (25 min).**
  RF: RF-2, RF-3. Dependencias: T09.
  Crear un estado nuevo con un hábito sin cumplimientos y rechazar duplicados.
  Hecho cuando: las pruebas comprueben creación sin marcas, conservación del nombre, rechazo de duplicados y ausencia de modificaciones en el estado recibido.

- [ ] **T11 — Implementar el cumplimiento diario en el núcleo (25 min).**
  RF: RF-2, RF-4, RF-5, RF-6. Dependencias: T10.
  Recibir una fecha explícita, localizar el hábito y distinguir entre marca nueva, repetida e inexistente.
  Hecho cuando: las pruebas demuestren una sola marca por fecha, repetición sin cambios, búsqueda normalizada y rechazo del hábito inexistente sin crearlo ni alterar entradas.

- [ ] **T12 — Implementar el cálculo básico de racha (25 min).**
  RF: RF-5, RF-7. Dependencias: T08.
  Aplicar el algoritmo del plan con fechas recibidas, sin consultar el reloj.
  Hecho cuando: pasen casos sin marcas, solo hoy, solo ayer, ausencia de ambos, secuencia consecutiva e interrupción por un día ausente; el orden y los duplicados de entrada no cambien el resultado.

- [ ] **T13 — Verificar límites del calendario (25 min).**
  RF: RF-6, RF-7. Dependencias: T12.
  Añadir las pruebas de cambios de mes, año, febrero y límites de fechas, ajustando únicamente el cálculo si fallan.
  Hecho cuando: pasen diciembre–enero, febrero bisiesto y no bisiesto y el límite inferior representable sin desbordamiento; se cubra la política aprobada en T02.

- [ ] **T14 — Preparar el listado desde el núcleo (25 min).**
  RF: RF-7, RF-8. Dependencias: T09, T12, T13.
  Obtener nombres conservados y rachas, ordenados según el criterio aprobado, sin entrada o salida.
  Hecho cuando: las pruebas verifiquen un resultado por hábito, rachas correctas, orden sin distinguir mayúsculas, colección vacía y entradas sin cambios, incluidos los casos de orden de T01.

## 3. Persistencia y protección de datos

- [ ] **T15 — Validar estructura y tipos del modelo JSON (25 min).**
  RF: RF-9, RF-10. Dependencias: T08.
  Validar objeto raíz, versión, campos y tipos según el contrato reconciliado en T07.
  Hecho cuando: el ejemplo del plan sea aceptado y versiones, campos o tipos no admitidos sean rechazados mediante pruebas, sin producir datos reparados silenciosamente.

- [ ] **T16 — Validar nombres y fechas persistidos (25 min).**
  RF: RF-2, RF-5, RF-6, RF-9, RF-10. Dependencias: T09, T15.
  Verificar unicidad de hábitos, validez del nombre y formato, validez y unicidad de fechas.
  Hecho cuando: las pruebas rechacen los casos inválidos acordados y acepten los registros válidos, incluido el resultado aprobado para fechas futuras existentes.

- [ ] **T17 — Implementar carga y primer uso (25 min).**
  RF: RF-9, RF-10. Dependencias: T15, T16.
  Leer desde una ruta controlable en pruebas y aplicar la política aprobada sobre datos ausentes.
  Hecho cuando: se recuperen hábitos y marcas del ejemplo; el primer uso produzca estado vacío sin crear archivos y el caso de desaparición respete T03.

- [ ] **T18 — Verificar fallos de lectura sin sobrescritura (20 min).**
  RF: RF-9, RF-10. Dependencias: T17.
  Cubrir archivo vacío, JSON mal formado, contenido inválido y acceso denegado controlado.
  Hecho cuando: cada caso produzca un fallo identificable, no se convierta en inicio vacío y el contenido anterior permanezca intacto.

- [ ] **T19 — Implementar guardado atómico e ida y vuelta (30 min).**
  RF: RF-9, RF-10. Dependencias: T16, T17.
  Validar, escribir UTF-8 en temporal del mismo directorio, sincronizar, cerrar y reemplazar el destino; crear el directorio en el primer guardado.
  Hecho cuando: guardar y cargar recupere exactamente los hábitos y fechas, los acentos se conserven y el destino no se trunque antes del reemplazo.

- [ ] **T20 — Verificar fallos anteriores al reemplazo (25 min).**
  RF: RF-9, RF-10. Dependencias: T19.
  Provocar fallos controlados al crear el directorio o temporal, escribir y sincronizar.
  Hecho cuando: cada fallo conserve el destino anterior, no se declare éxito y la limpieza no oculte la causa original.

- [ ] **T21 — Verificar reemplazo fallido y recuperación (25 min).**
  RF: RF-9, RF-10. Dependencias: T19, T20.
  Provocar fallo del reemplazo y comprobar una nueva lectura después de los puntos de interrupción acordados.
  Hecho cuando: las pruebas acrediten las garantías concretas de T05 sin presentar datos parciales como válidos ni prometer garantías superiores a las acordadas.

## 4. Contrato y coordinación de la CLI

- [ ] **T22 — Definir argumentos y ayuda en español (25 min).**
  RF: RF-1, RF-11. Dependencias: T08.
  Preparar los tres comandos, sus argumentos y la ayuda global y por comando.
  Hecho cuando: las pruebas de ayuda enumeren `create`, `done` y `list`, describan sus argumentos en español y terminen con código 0 sin leer datos.

- [ ] **T23 — Rechazar entradas de CLI inválidas (25 min).**
  RF: RF-1, RF-2, RF-6, RF-11. Dependencias: T09, T22.
  Gestionar comando ausente o desconocido, nombre ausente o vacío, argumentos adicionales y fechas no admitidas.
  Hecho cuando: cada caso devuelva código 2 por error estándar con orientación de uso, sin acceso de escritura; un nombre con espacios y otro iniciado por guion se acepten mediante la sintaxis del plan.

- [ ] **T24 — Capturar y transmitir la fecha local (20 min).**
  RF: RF-4, RF-6, RF-7. Dependencias: T11, T14, T22.
  Capturar una sola referencia al inicio y transmitirla a las operaciones que la requieren.
  Hecho cuando: una prueba con reloj controlado demuestre que un cambio de día durante el comando no altera la fecha usada por la operación.

- [ ] **T25 — Traducir fallos de dominio y persistencia (25 min).**
  RF: RF-3, RF-4, RF-10, RF-11. Dependencias: T10, T11, T18, T21, T23.
  Centralizar la presentación de errores esperables sin trasladar mensajes de consola al núcleo.
  Hecho cuando: duplicado, inexistente, lectura inválida y guardado fallido produzcan los mensajes acordados en español, código 1, error estándar y ninguna traza ni confirmación de éxito.

- [ ] **T26 — Conectar el comando de creación (25 min).**
  RF: RF-2, RF-3, RF-9, RF-10, RF-11. Dependencias: T10, T19, T22, T23, T25.
  Cargar, crear, guardar y confirmar, en ese orden.
  Hecho cuando: la prueba de integración cree un hábito sin marcas recuperable en otra carga y no confirme éxito ante un guardado fallido; el duplicado conserve los datos.

- [ ] **T27 — Conectar el comando de cumplimiento (25 min).**
  RF: RF-2, RF-4, RF-5, RF-6, RF-9, RF-10, RF-11. Dependencias: T11, T19, T24, T25, T26.
  Registrar la fecha capturada, guardar solo cambios y distinguir repetición de error.
  Hecho cuando: la marca nueva se conserve antes de confirmarse; repetir devuelva 0 y el mensaje exacto sin guardar, y un hábito inexistente devuelva 1 sin cambios.

- [ ] **T28 — Conectar y presentar el listado (25 min).**
  RF: RF-7, RF-8, RF-9, RF-10, RF-11. Dependencias: T14, T17, T24, T25.
  Mostrar la cabecera y las filas del contrato o el mensaje de colección vacía.
  Hecho cuando: las pruebas verifiquen contenido, orden, rachas y código 0; ni el listado vacío ni el listado con hábitos creen o modifiquen archivos.

- [ ] **T29 — Conectar el punto de entrada del paquete (20 min).**
  RF: RF-1, RF-11. Dependencias: T26, T27, T28.
  Delegar `python -m habits` en la CLI y transmitir su código de salida.
  Hecho cuando: pruebas en procesos separados comprueben código 0 con ayuda, 2 con comando inválido y 1 con un fallo de dominio, usando almacenamiento temporal aislado.

- [ ] **T30 — Comprobar prioridades de ayuda y errores (20 min).**
  RF: RF-1, RF-10, RF-11. Dependencias: T25, T29.
  Probar ayuda e invocaciones inválidas cuando los datos guardados estén dañados.
  Hecho cuando: la ayuda termine con 0 sin leerlos, la entrada inválida termine con 2 antes de leerlos y un comando válido informe del problema de datos con 1 sin sobrescritura.

## 5. Aceptación, documentación y cierre

- [ ] **T31 — Verificar creación y listado entre procesos (25 min).**
  RF: RF-1, RF-2, RF-3, RF-8, RF-9. Dependencias: T29.
  Ejecutar listado inicial, creación, nueva ejecución del listado y creación duplicada con variantes de nombre.
  Hecho cuando: el flujo real muestre inicialmente el mensaje vacío, después el nombre conservado con racha 0 y finalmente un duplicado rechazado sin cambios; todos los procesos usen datos temporales.

- [ ] **T32 — Verificar cumplimiento y repetición entre procesos (25 min).**
  RF: RF-4, RF-5, RF-6, RF-7, RF-8, RF-9. Dependencias: T29, T31.
  Ejecutar marca, listado y repetición en procesos independientes con una referencia temporal controlada.
  Hecho cuando: el listado muestre racha 1, la repetición termine con 0 y el mensaje exacto, y el contenido persistido sea idéntico antes y después de repetir.

- [ ] **T33 — Verificar rachas al cambiar el día (25 min).**
  RF: RF-6, RF-7, RF-8, RF-9. Dependencias: T13, T24, T28, T32.
  Preparar cumplimientos de los días 23 y 24 y consultar con referencias controladas del 25 y 26; comprobar dos ejecuciones a ambos lados de medianoche.
  Hecho cuando: las consultas muestren respectivamente 2 y 0, cada ejecución mantenga su propia referencia y no exista una opción pública para alterar la fecha.

- [ ] **T34 — Verificar integridad desde los comandos (25 min).**
  RF: RF-3, RF-4, RF-5, RF-8, RF-9, RF-10, RF-11. Dependencias: T20, T21, T26, T27, T28, T30.
  Comprobar duplicado, inexistente, repetición, listado, datos corruptos y fallos de guardado comparando el destino antes y después.
  Hecho cuando: rechazos, consultas y repetición conserven los bytes guardados, los fallos de escritura conserven el destino previo y ninguna operación fallida confirme éxito.

- [ ] **T35 — Documentar el uso y revisar los mensajes (25 min).**
  RF: RF-1, RF-3, RF-4, RF-5, RF-8, RF-10, RF-11. Dependencias: T06, T29, T30.
  Actualizar la documentación de uso con ejemplos de los tres comandos, ayuda, códigos de salida y errores habituales.
  Hecho cuando: los ejemplos coincidan con el comportamiento comprobado, todos los mensajes estén en español y la ayuda y errores cumplan el criterio acordado en T06.

- [ ] **T36 — Auditar la trazabilidad y la constitución (25 min).**
  RF: RF-1 a RF-11. Dependencias: T07, T31, T32, T33, T34, T35 y cualquier tarea adicional derivada de T07.
  Relacionar cada criterio de aceptación con su prueba y revisar separación de responsabilidades, tipos, idioma, dependencias y alcance.
  Hecho cuando: cada criterio tenga evidencia identificable, no queden dudas bloqueantes ni ampliaciones sin aprobar, y se confirme que las pruebas no usan datos personales.

- [ ] **T37 — Ejecutar la verificación final y registrar el resultado (20 min).**
  RF: RF-1 a RF-11. Dependencias: T36.
  Ejecutar `pytest -q` y registrar el resultado junto con la revisión de los criterios de finalización de la especificación.
  Hecho cuando: todas las pruebas pasen y cada criterio de finalización tenga evidencia; si existen fallos, registrar tareas de corrección de hasta 30 minutos y dejar esta tarea sin marcar hasta que se resuelvan y se repita la verificación.

## Trazabilidad resumida

Esta tabla señala las tareas principales; los RF detallados en cada tarea siguen
siendo la referencia para sus dependencias y verificaciones.

| RF | Tareas principales |
| --- | --- |
| RF-1 | T22, T23, T29, T30, T31, T35 |
| RF-2 | T01, T09, T10, T11, T16, T23, T26, T27, T31 |
| RF-3 | T01, T10, T25, T26, T31, T34 |
| RF-4 | T11, T24, T25, T27, T32, T34 |
| RF-5 | T11, T12, T16, T27, T32, T34 |
| RF-6 | T02, T11, T13, T16, T23, T24, T27, T32, T33 |
| RF-7 | T02, T12, T13, T14, T24, T28, T32, T33 |
| RF-8 | T01, T14, T28, T31, T32, T33, T34, T35 |
| RF-9 | T03, T04, T05, T15 a T21, T26 a T28, T31 a T34 |
| RF-10 | T03, T04, T05, T15 a T21, T25 a T28, T30, T34, T35 |
| RF-11 | T06, T22, T23, T25 a T30, T34, T35 |

T07, T36 y T37 cubren todos los RF como controles documentales y de cierre.

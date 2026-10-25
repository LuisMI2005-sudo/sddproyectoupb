# Especificación 001: MVP de habits-cli

## Contexto y objetivo

Un estudiante necesita registrar el cumplimiento diario de sus hábitos de estudio
y consultar su continuidad de manera sencilla. El MVP permite crear hábitos,
marcarlos como hechos hoy y listar su racha actual de días consecutivos mediante
una interfaz de línea de comandos.

El objetivo es facilitar el seguimiento personal sin exigir cuentas ni registros
de fechas manuales. La racha de ayer se conserva durante el día actual para no
penalizar al estudiante antes de que termine su oportunidad de estudiar hoy.

## Usuarios

- **Estudiante:** administra sus propios hábitos y consulta su constancia desde
  el mismo equipo. No necesita iniciar sesión ni compartir sus datos.

## Historias de usuario

- **HU-1:** Como estudiante, quiero crear un hábito con un nombre reconocible para
  identificar la actividad de estudio que deseo mantener.
- **HU-2:** Como estudiante, quiero marcar un hábito como hecho hoy para registrar
  mi cumplimiento diario sin introducir fechas.
- **HU-3:** Como estudiante, quiero listar mis hábitos y sus rachas actuales para
  conocer mi continuidad de estudio.
- **HU-4:** Como estudiante, quiero conservar mis registros entre ejecuciones y
  recibir errores comprensibles para usar la aplicación sin perder información.

## Requisitos funcionales

### RF-1. Operaciones disponibles y ayuda

La aplicación ofrecerá `create <nombre>`, `done <nombre>` y `list`, además de ayuda
de uso, para que el estudiante pueda descubrir y realizar las tres operaciones.

**Criterios de aceptación:**

- Siempre, el sistema deberá ofrecer las tres operaciones con los nombres y
  argumentos indicados.
- CUANDO el usuario consulte la ayuda, el sistema deberá describir en español
  las operaciones disponibles y sus argumentos obligatorios.

### RF-2. Identificación única por nombre

Cada hábito se identificará por su nombre, sin espacios en los extremos y sin
distinguir mayúsculas, para evitar duplicados por diferencias de presentación.

**Criterios de aceptación:**

- Siempre, el sistema deberá comparar los nombres sin distinguir mayúsculas y
  después de quitar sus espacios iniciales y finales.
- CUANDO el usuario cree o busque un hábito con espacios en los extremos, el
  sistema deberá utilizar el nombre sin esos espacios.
- SI el nombre está vacío o solo contiene espacios, ENTONCES el sistema deberá
  rechazar la operación con un error claro y conservar los datos existentes.

### RF-3. Creación de hábitos

El usuario podrá crear un hábito para comenzar su seguimiento sin registrar
automáticamente un cumplimiento.

**Criterios de aceptación:**

- CUANDO el usuario ejecute `create <nombre>` con un nombre válido y no existente,
  el sistema deberá crear el hábito sin marcas de cumplimiento y confirmar su
  creación en español.
- CUANDO se cree un hábito, el sistema deberá conservar las mayúsculas del nombre
  introducido, sin los espacios de los extremos, para mostrarlo posteriormente.
- CUANDO el usuario consulte un hábito recién creado, el sistema deberá mostrar
  una racha actual de cero días.
- SI ya existe un hábito con el mismo nombre según RF-2, ENTONCES el sistema
  deberá rechazar la creación, explicar que ya existe y no modificar los datos.

### RF-4. Registro del cumplimiento de hoy

El usuario podrá marcar un hábito existente como hecho hoy para registrar su
actividad diaria.

**Criterios de aceptación:**

- CUANDO el usuario ejecute `done <nombre>` para un hábito existente que todavía
  no esté marcado hoy, el sistema deberá registrar su cumplimiento de hoy y
  mostrar una confirmación clara en español.
- SI el hábito indicado no existe, ENTONCES el sistema deberá mostrar un error
  claro, no crear el hábito y no modificar los datos.
- CUANDO el usuario identifique el hábito con otras mayúsculas o espacios en los
  extremos, el sistema deberá resolverlo mediante las reglas de RF-2 sin cambiar
  su nombre de presentación.

### RF-5. Repetición de una marca diaria

Cada hábito contará como cumplido una sola vez por día para impedir que las
repeticiones aumenten artificialmente su racha.

**Criterios de aceptación:**

- SI el hábito ya está marcado hoy, ENTONCES el sistema deberá mostrar «Ya estaba
  marcado como hecho hoy» y considerar la operación exitosa sin modificar datos.
- Siempre, el sistema deberá contar como máximo un cumplimiento por hábito y
  fecha al calcular la racha.

### RF-6. Fecha de referencia

El registro se limitará al día actual para simplificar el seguimiento diario.

**Criterios de aceptación:**

- CUANDO comience la ejecución de un comando, el sistema deberá tomar como «hoy»
  la fecha local del equipo y utilizarla como referencia durante esa ejecución.
- Siempre, el sistema deberá permitir registrar únicamente el cumplimiento del
  día actual, sin ofrecer una fecha alternativa.
- SI se intenta indicar una fecha pasada o futura como argumento, ENTONCES el
  sistema deberá rechazar la entrada como inválida sin modificar datos.
- SI un retroceso del reloj o un cambio de zona horaria deja cumplimientos
  existentes en fechas posteriores a hoy, ENTONCES el sistema deberá conservarlos
  sin rechazarlos como datos dañados por ese motivo ni impedir el registro de hoy.
- CUANDO se marque hoy después de ese cambio de fecha, el sistema deberá aplicar
  las mismas reglas de registro y repetición diaria de RF-4 y RF-5, sin borrar ni
  trasladar los cumplimientos de otras fechas.

### RF-7. Cálculo de la racha actual

La racha mostrará el número de días consecutivos de cumplimiento que llegan hasta
hoy o ayer, de modo que un día todavía en curso no interrumpa por sí solo la racha.

**Criterios de aceptación:**

- CUANDO exista un cumplimiento de hoy, el sistema deberá mostrar el número de
  días consecutivos con cumplimiento que terminan hoy.
- MIENTRAS hoy no tenga un cumplimiento y ayer sí lo tenga, el sistema deberá
  mostrar el número de días consecutivos con cumplimiento que terminan ayer.
- SI no existen cumplimientos ni hoy ni ayer, ENTONCES el sistema deberá mostrar
  una racha actual de cero días.
- SI el hábito no tiene cumplimientos, ENTONCES el sistema deberá mostrar una
  racha actual de cero días.
- Siempre, el sistema deberá contar días de calendario consecutivos, incluidos
  cambios de mes y año y el día adicional de los años bisiestos.
- Siempre, el sistema deberá detener el conteo de una racha al encontrar un día
  sin cumplimiento, sin sumar segmentos anteriores separados por esa ausencia.
- MIENTRAS existan cumplimientos posteriores a la fecha de referencia, el sistema
  deberá excluirlos del cálculo de la racha actual sin eliminarlos; cuando esas
  fechas dejen de ser futuras, deberán participar en el cálculo según las reglas
  ordinarias de racha.

### RF-8. Listado de hábitos

El listado permitirá revisar todos los hábitos y comparar su continuidad actual.

**Criterios de aceptación:**

- CUANDO el usuario ejecute `list` y existan hábitos, el sistema deberá mostrar
  cada hábito una sola vez, con su nombre de presentación y su racha actual en días.
- Siempre, el sistema deberá ordenar el listado alfabéticamente por nombre sin
  distinguir mayúsculas.
- CUANDO el usuario ejecute `list` sin hábitos registrados, el sistema deberá
  mostrar «No hay hábitos registrados» y considerar la operación exitosa.
- CUANDO el usuario consulte el listado, el sistema deberá conservar los datos
  sin crear hábitos ni registrar cumplimientos.

### RF-9. Conservación entre ejecuciones

Los hábitos y sus cumplimientos permanecerán disponibles para continuar el
seguimiento después de cerrar la aplicación.

**Criterios de aceptación:**

- CUANDO una creación o un cumplimiento se confirme como exitoso, el sistema
  deberá conservarlo para las siguientes ejecuciones.
- CUANDO se utilice la aplicación por primera vez, el sistema deberá permitir
  comenzar sin hábitos registrados.
- CUANDO el usuario vuelva a utilizar la aplicación, el sistema deberá recuperar
  los hábitos y cumplimientos conservados y calcular las rachas con la nueva fecha
  de referencia.

### RF-10. Protección ante problemas con los datos

La aplicación priorizará conservar los registros anteriores ante un problema de
lectura, de guardado o de integridad de los datos.

**Criterios de aceptación:**

- SI los datos guardados están dañados o no se pueden leer, ENTONCES el sistema
  deberá informar del problema, indicar que la operación falló y no sobrescribir
  ni reemplazar esos datos por un conjunto vacío.
- SI una operación no puede guardar sus cambios, ENTONCES el sistema deberá
  informar del fallo, conservar los datos anteriores y no confirmar el cambio
  como exitoso.
- Siempre, el sistema deberá evitar que un guardado fallido deje los registros
  anteriores parcialmente modificados.

### RF-11. Errores de uso y resultado de las operaciones

El usuario recibirá explicaciones comprensibles para corregir entradas inválidas
y distinguir operaciones exitosas de fallidas.

**Criterios de aceptación:**

- SI el comando es desconocido o sus argumentos son inválidos o incompletos,
  ENTONCES el sistema deberá rechazar la operación sin modificar datos, explicar
  el problema e indicar el uso correcto.
- Siempre, el sistema deberá mostrar los mensajes dirigidos al usuario en español
  y evitar trazas técnicas en los errores contemplados en esta especificación.
- CUANDO una operación termine, el sistema deberá indicar un resultado de éxito
  o fallo coherente con lo ocurrido.

## Requisitos no funcionales

- **RNF-1. Comprensibilidad:** la ayuda, confirmaciones y errores serán breves,
  claros y comprensibles para un estudiante sin conocimientos técnicos.
- **RNF-2. Consistencia:** con los mismos hábitos, cumplimientos y fecha de
  referencia, el listado y las rachas tendrán el mismo resultado.
- **RNF-3. Integridad:** una operación rechazada o fallida no deberá perder,
  duplicar ni modificar registros previamente conservados.
- **RNF-4. Autonomía:** el seguimiento personal no requerirá cuentas ni conexión
  con servicios externos.
- **RNF-5. Verificabilidad:** cada regla funcional tendrá pruebas de aceptación
  vinculadas a su requisito, incluidos los errores y casos límite pertinentes.

## Casos límite

| Caso | Resultado esperado |
| --- | --- |
| Nombre vacío o compuesto solo por espacios | Error sin cambios. |
| Crear «Leer» y después « leer » | La segunda creación falla por duplicado. |
| Crear «  Leer Matemáticas  » | Se muestra «Leer Matemáticas», con racha cero. |
| Marcar «LEER» cuando existe «Leer» | Se marca el hábito existente sin renombrarlo. |
| Marcar un hábito inexistente | Error; no se crea ningún hábito. |
| Marcar varias veces el mismo día | Una sola marca; las repeticiones son exitosas sin cambios. |
| Hábito sin cumplimientos | Racha cero. |
| Marcas el 23 y 24 de octubre; consulta el 25 sin marca de hoy | Racha de dos días. |
| Las mismas marcas; consulta el 26 sin nuevas marcas | Racha cero. |
| Marcas ayer y hoy, con un día anterior ausente | Racha de dos días; no se suman marcas más antiguas. |
| Cumplimientos el 31 de diciembre y el 1 de enero; consulta el 1 | Racha de dos días. |
| Cumplimientos el 28 y 29 de febrero y el 1 de marzo de un año bisiesto; consulta el 1 | Racha de tres días. |
| Dos ejecuciones a distintos lados de la medianoche local | Cada ejecución usa su propio día de referencia. |
| Marcas del 24 y 25 de octubre de 2026; el reloj retrocede al 24 | La consulta muestra racha 1 y conserva ambas marcas. Marcar el 24 de nuevo informa «Ya estaba marcado como hecho hoy» y no modifica datos. |
| Solo hay marca del 25 de octubre de 2026; un cambio de zona horaria sitúa hoy en el 24 | La consulta muestra racha 0 y conserva la marca del 25. Marcar hoy añade el 24; una consulta posterior ese día muestra racha 1 y conserva ambas fechas. |
| Listado sin hábitos | Mensaje «No hay hábitos registrados» y resultado exitoso. |
| Primer uso sin datos previos | Inicio sin hábitos; crear un hábito está permitido. |
| Datos guardados dañados o inaccesibles | Error sin tratarlos como un primer uso ni sobrescribirlos. |
| Fallo al guardar | Error sin confirmación de éxito y conservación de los datos anteriores. |
| Comando desconocido, argumento ausente o adicional | Error con orientación de uso, sin cambios. |

## Fuera de alcance

- Eliminar o renombrar hábitos.
- Deshacer o editar cumplimientos.
- Registrar fechas pasadas o futuras.
- Consultar el historial o la mejor racha.
- Recordatorios y notificaciones.
- Cuentas, autenticación, uso compartido y sincronización.
- Exportación de datos.
- Interfaces gráficas.

## Criterios de finalización

- Los tres comandos y la ayuda cumplen todos los criterios de aceptación de
  RF-1 a RF-11.
- Los requisitos no funcionales y los casos límite descritos están verificados.
- Cada regla funcional cuenta con pruebas pertinentes y todas pasan antes de
  considerar terminada la funcionalidad.
- Se verifica la conservación de hábitos y cumplimientos entre ejecuciones,
  incluido el comportamiento ante fallos de lectura y guardado.
- El uso de los comandos y sus errores está documentado en español.
- No se incorporan funciones declaradas fuera del alcance.
- No quedan dudas abiertas que impidan verificar el comportamiento acordado.

## Dudas abiertas

La duda sobre retrocesos del reloj y cambios de zona horaria quedó resuelta por
aprobación explícita del usuario en T02. Sus reglas y los dos escenarios aprobados
están incorporados en RF-6, RF-7 y los casos límite.

Las demás aclaraciones detectadas en la revisión QA siguen registradas en la
sección 5 del plan y en sus tareas correspondientes; T02 no las resuelve.
Cualquier nueva ambigüedad que afecte al comportamiento deberá registrarse con
la marca **[NECESITA ACLARACIÓN]** y resolverse antes de implementar el requisito
afectado.

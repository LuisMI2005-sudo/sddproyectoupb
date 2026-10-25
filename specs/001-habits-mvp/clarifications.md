# Aclaraciones y decisiones — MVP de habits-cli

Este registro conserva las respuestas explícitas del usuario que dieron origen
a la [especificación](spec.md). No sustituye sus criterios de aceptación ni
considera aprobadas las propuestas técnicas del [plan](plan.md).

## Preguntas iniciales aprobadas

### D-01. Identificación de hábitos

- **Pregunta:** ¿cómo se identifica cada hábito y se evitan duplicados?
- **Decisión aprobada:** usar el nombre sin espacios en los extremos y sin
  distinguir mayúsculas. No aceptar nombres vacíos. Rechazar la creación de un
  duplicado con un error claro, sin modificar los datos.
- **Ejemplo:** crear «Leer» y después « leer » produce un solo hábito; el segundo
  intento falla.
- **Propósito:** evitar duplicados por diferencias de presentación.
- **RF:** RF-2 y RF-3.

### D-02. Racha actual y repetición diaria

- **Pregunta:** ¿cómo se cuenta la racha y qué ocurre al repetir una marca hoy?
- **Decisión aprobada:** contar días consecutivos hasta hoy o ayer. Sin marca en
  ninguno, la racha es cero. Un hábito nuevo empieza con cero. Repetir hoy cuenta
  una sola vez y muestra «Ya estaba marcado como hecho hoy», sin cambios.
- **Ejemplo:** marcas del 23 y 24, consultadas el 25 sin marcar hoy, dan racha 2.
- **Propósito:** no romper la racha antes de terminar el día en curso y evitar
  incrementos artificiales por repetir la operación.
- **RF:** RF-3, RF-5 y RF-7.

### D-03. Significado de hoy

- **Pregunta:** ¿qué fecha se utiliza y se pueden registrar otras fechas?
- **Decisión aprobada:** usar la fecha local del equipo. Registrar únicamente hoy;
  no admitir fechas pasadas ni futuras como entrada. Contar días de calendario,
  incluidos cambios de mes y año.
- **Ejemplo:** el día siguiente al 31 de diciembre es el 1 de enero y continúa
  una racha si ambos días tienen cumplimiento.
- **Propósito:** simplificar el registro diario sin pedir fechas manuales.
- **RF:** RF-4, RF-6 y RF-7.

### D-04. Contenido del listado

- **Pregunta:** ¿qué se muestra y en qué orden?
- **Decisión aprobada:** mostrar nombre y racha actual en días, en orden
  alfabético sin distinguir mayúsculas. Conservar las mayúsculas originales del
  nombre, quitando sus espacios exteriores. Sin hábitos, mostrar «No hay hábitos
  registrados». Crear no registra automáticamente un cumplimiento.
- **Ejemplo:** «  Leer Matemáticas  » se muestra como «Leer Matemáticas» con
  racha inicial cero.
- **Propósito:** ofrecer una consulta consistente de todas las actividades.
- **RF:** RF-3 y RF-8.

### D-05. Conservación y errores

- **Pregunta:** ¿qué ocurre entre ejecuciones y ante entradas o datos inválidos?
- **Decisión aprobada:** conservar hábitos y marcas entre ejecuciones. Empezar
  sin hábitos en el primer uso. No crear un hábito al intentar marcar uno
  inexistente. Explicar errores de argumentos; ante fallos de lectura o guardado,
  informar sin perder ni sobrescribir datos anteriores. Mostrar mensajes en
  español sin trazas técnicas. Una marca repetida es exitosa sin cambios.
- **Ejemplo:** marcar un hábito inexistente falla sin añadirlo; marcar hoy por
  segunda vez es exitoso y no duplica el cumplimiento.
- **Propósito:** proteger los registros y permitir comprender los resultados.
- **RF:** RF-4, RF-5, RF-9, RF-10 y RF-11.

### D-06. Comandos y límites del MVP

- **Pregunta:** ¿cuáles son los comandos y qué queda fuera?
- **Decisión aprobada:** `create <nombre>`, `done <nombre>` y `list`, con ayuda y
  mensajes en español. Confirmar las creaciones y primeras marcas exitosas.
  Excluir eliminar, renombrar, deshacer, historial, mejor racha, otras fechas,
  recordatorios, cuentas, sincronización y exportación.
- **Ejemplo:** una petición de registrar ayer no pertenece al MVP y no puede
  modificar datos.
- **Propósito:** acotar la primera funcionalidad a un seguimiento diario simple.
- **RF:** RF-1, RF-3, RF-4, RF-6 y RF-11.

## Aclaración posterior aprobada: T02

### D-07. Retrocesos del reloj y cambios de zona horaria

- **Estado:** aprobado explícitamente por el usuario; T02 completada.
- **Decisión:** conservar los cumplimientos que pasan a ser futuros. No cuentan
  para la racha actual mientras sean futuros y no impiden marcar la nueva fecha
  local de hoy. Las marcas ya existentes de hoy se tratan como repeticiones.
- **RF:** RF-6 y RF-7, aplicando las reglas de registro de RF-4 y RF-5.

| Escenario aprobado | Consulta | Registro de hoy |
| --- | --- | --- |
| Marcas del 24 y 25 de octubre de 2026; el reloj retrocede al 24. | Racha 1; ambas marcas se conservan. | El 24 ya está marcado: repetición exitosa sin cambios. |
| Solo hay marca del 25 de octubre de 2026; la nueva zona horaria sitúa hoy en el 24. | Racha 0; se conserva la marca del 25. | Se añade el 24; al consultar ese día, la racha es 1 y se conservan ambas marcas. |

La aprobación quedó incorporada en la especificación y el plan. No supone que
se haya implementado o probado el comportamiento.

## Dudas que continúan abiertas

| Tarea | RF o RNF afectados | Aclaración pendiente |
| --- | --- | --- |
| T01 | RF-2, RF-3, RF-8 | [NECESITA ACLARACIÓN] Caracteres admitidos, longitud, equivalencia Unicode y orden lingüístico. |
| T03 | RF-9, RF-10 | [NECESITA ACLARACIÓN] Datos desaparecidos después del primer uso. |
| T04 | RF-9, RF-10 | [NECESITA ACLARACIÓN] Uso simultáneo desde varios procesos. |
| T05 | RF-9, RF-10 | [NECESITA ACLARACIÓN] Garantías ante interrupción abrupta o corte de energía. |
| T06 | RF-1, RF-11, RNF-1 | [NECESITA ACLARACIÓN] Criterio de comprensibilidad de ayuda y mensajes. |

Los detalles de contrato y validación propuestos en el plan siguen sujetos a la
reconciliación documental de T07. Este registro no resuelve esas tareas por
suposición ni altera su estado.

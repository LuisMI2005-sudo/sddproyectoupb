# Plan técnico: MVP de habits-cli

Documento de planificación basado en la constitución y en la especificación 001.
No contiene implementación. Las decisiones técnicas propuestas se distinguen de
las dudas de producto pendientes, recogidas en la sección 5.

## 1. Estructura de módulos

**Cobertura: RF-1 a RF-11.**

| Módulo previsto | Responsabilidad | RF cubiertos |
| --- | --- | --- |
| `habits/__init__.py` | Identificar el paquete, sin ejecutar operaciones al importarlo. | RF-1, soporte |
| `habits/__main__.py` | Delegar la ejecución en la CLI y devolver su código de salida. | RF-1, RF-11 |
| `habits/core.py` | Normalizar nombres, crear hábitos, registrar cumplimientos, ordenar hábitos y calcular rachas mediante funciones puras. | RF-2, RF-3, RF-4, RF-5, RF-7, RF-8 |
| `habits/storage.py` | Leer, validar y guardar los datos locales sin imprimir mensajes ni decidir fechas. | RF-9, RF-10 |
| `habits/cli.py` | Interpretar argumentos, tomar la fecha local una sola vez al inicio, coordinar núcleo y persistencia, y mostrar resultados en español. | RF-1, RF-2, RF-3, RF-4, RF-5, RF-6, RF-8, RF-9, RF-10, RF-11 |
| `tests/test_core.py` | Verificar reglas de nombres, hábitos, marcas y rachas con fechas explícitas. | RF-2 a RF-8 |
| `tests/test_storage.py` | Verificar recuperación, validación y protección de datos. | RF-9, RF-10 |
| `tests/test_cli.py` | Verificar comandos, ayuda, mensajes y códigos de salida. | RF-1 a RF-6, RF-8 a RF-11 |
| `tests/test_acceptance.py` | Verificar recorridos completos y conservación entre ejecuciones. | RF-1 a RF-11 |

El núcleo recibe los datos y la fecha de referencia como entradas, y devuelve
resultados sin modificar sus entradas. No importa la CLI ni la persistencia, no
consulta el reloj y no realiza entrada o salida. La CLI depende del núcleo y del
almacenamiento; estos no dependen de ella.

La creación y el registro producen un estado nuevo; una repetición diaria no
produce cambios. La CLI guarda únicamente las operaciones que cambian el estado
y confirma su éxito después de completar el guardado. El listado nunca guarda.

Las funciones públicas tendrán anotaciones de tipos. Los identificadores serán
ingleses; la presentación y los errores dirigidos al usuario serán españoles.

## 2. Modelo de datos JSON con ejemplo

**Cobertura: RF-2, RF-3, RF-4, RF-5, RF-6, RF-7, RF-8, RF-9 y RF-10.**

Se propone un objeto raíz con una versión de esquema y una lista de hábitos.

| Campo | Tipo | Regla |
| --- | --- | --- |
| `schema_version` | Entero | Valor inicial 1; otras versiones no se interpretan ni sobrescriben. |
| `habits` | Lista | Puede estar vacía. |
| `name` | Texto | Nombre de presentación, no vacío y sin espacios en los extremos; conserva sus mayúsculas. |
| `completed_dates` | Lista de textos | Fechas de calendario válidas en formato exacto `AAAA-MM-DD`, sin duplicados dentro del hábito. |

Ejemplo ilustrativo de datos, no código de implementación:

```json
{
  "schema_version": 1,
  "habits": [
    {
      "name": "Leer Matemáticas",
      "completed_dates": ["2026-10-23", "2026-10-24"]
    },
    {
      "name": "Repasar Programación",
      "completed_dates": []
    }
  ]
}
```

Con referencia al 25 de octubre de 2026, las rachas del ejemplo son 2 y 0.
Las fechas ilustran registros acumulados en días anteriores: no habilitan un
comando para introducir fechas manualmente.

El identificador de búsqueda se deriva del nombre al quitar espacios exteriores
y aplicar comparación sin mayúsculas. No se almacena una segunda copia del
identificador ni se persiste la racha: ambos se calculan para evitar discrepancias.

La validación propuesta comprueba estructura, campos obligatorios, tipos,
versión, nombres, unicidad de hábitos según RF-2 y validez y unicidad de fechas.
Una estructura no reconocida se rechaza sin intentar repararla. Los campos
desconocidos también se rechazan para evitar perder información al reescribir.
Los registros se validan antes de usarse y antes de guardarse.

Ubicación propuesta: `habits.json` en el directorio `.habits-cli` del directorio
personal del usuario. Así se conservan los mismos datos al cambiar el directorio
de trabajo. La ruta se podrá proporcionar internamente en pruebas; no se añade
un argumento público de configuración al MVP.

En un primer uso sin archivo de datos se prepara un estado vacío en memoria.
La consulta no crea archivos. Al guardar por primera vez se crea el directorio
si hace falta; cualquier fallo se comunica como error de persistencia.

Guardado previsto:

1. Validar el estado nuevo y serializarlo como JSON UTF-8 legible.
2. Escribirlo en un temporal del mismo directorio que el destino.
3. Vaciar los búferes, sincronizar el contenido y cerrar el temporal.
4. Reemplazar el destino de forma atómica; nunca truncarlo previamente.
5. Ante un fallo previo al reemplazo, conservar el destino y no confirmar éxito.
   La limpieza de temporales no debe ocultar el error original.

Un archivo vacío, ilegible o inválido no equivale a un primer uso. No se reinicia
el estado ni se sobrescribe ese archivo. Las migraciones no se realizarán
automáticamente; cualquier cambio posterior de esquema requiere actualizar antes
la especificación, como exige la constitución.

## 3. Algoritmo de cálculo de racha en pseudocódigo

**Cobertura: RF-5, RF-6 y RF-7; proporciona el valor mostrado por RF-8.**

Entradas: cumplimientos de un hábito y fecha local de referencia capturada por la
CLI al inicio. Salida: entero no negativo. Las fechas se tratan como días de
calendario, sin convertir la separación entre días en horas transcurridas.

```text
CALCULAR RACHA ACTUAL(cumplimientos, fecha de referencia)
    Construir un conjunto de fechas distintas a partir de los cumplimientos.

    SI la fecha de referencia está en el conjunto:
        Elegir esa fecha como día de partida.
    EN OTRO CASO, SI el día anterior a la referencia está en el conjunto:
        Elegir ese día anterior como día de partida.
    EN OTRO CASO:
        Devolver cero.

    Iniciar el contador en cero.
    MIENTRAS el día elegido esté en el conjunto:
        Aumentar el contador en uno.
        Retroceder al día anterior del calendario.
        Si se alcanza el límite inferior de fechas representables, terminar.

    Devolver el contador.
```

El conjunto impide contar varias veces el mismo día y evita depender del orden
de entrada. Un día ausente detiene el conteo. El retroceso por calendario cubre
cambios de mes y año y años bisiestos. Para n fechas registradas y una racha de
k días, el coste previsto es O(n + k) en tiempo y O(n) en memoria.

## 4. Contrato de la CLI

**Cobertura: RF-1 a RF-11.**

Entrada común: `python -m habits <comando>`. Un nombre con espacios se proporciona
como un único argumento entre comillas. No se aceptan fechas ni argumentos
adicionales en los tres comandos. Los nombres que comiencen con guion podrán
pasarse después del separador de opciones `--`.

| Invocación | Salida normal propuesta | Efecto | RF |
| --- | --- | --- | --- |
| `python -m habits create "Leer"` | `Hábito creado: Leer.` | Crea el hábito sin cumplimientos, después de guardar. | RF-2, RF-3, RF-9 |
| `python -m habits done "Leer"` | `Hábito marcado como hecho hoy: Leer.` | Añade únicamente la fecha local capturada al inicio y guarda. | RF-2, RF-4, RF-6, RF-9 |
| La misma marca repetida hoy | `Ya estaba marcado como hecho hoy` | No modifica ni vuelve a guardar los datos. | RF-5 |
| `python -m habits list` con hábitos | Cabecera `Hábito | Racha (días)` y una fila `<nombre> | <racha>` por hábito. | Ordena para mostrar y calcula rachas; no guarda. | RF-7, RF-8 |
| `python -m habits list` sin hábitos | `No hay hábitos registrados` | No modifica ni crea datos. | RF-8, RF-9 |
| `python -m habits --help` o ayuda de un comando con `--help` | Uso, descripción, comandos y argumentos en español. | No lee ni modifica datos. | RF-1, RF-11 |

Las confirmaciones y filas utilizan el nombre de presentación conservado, incluso
cuando la búsqueda se haya hecho con otras mayúsculas. No se añaden colores ni
dependencias para presentar el listado.

| Código de salida | Situaciones | Canal de salida | RF |
| --- | --- | --- | --- |
| 0 | Creación o marca guardada, marca repetida, listado y ayuda solicitada. | Salida estándar. | RF-1, RF-3, RF-4, RF-5, RF-8, RF-9 |
| 1 | Hábito duplicado, hábito inexistente, datos inválidos o fallo de lectura o guardado. | Error estándar; sin confirmación de éxito. | RF-3, RF-4, RF-10, RF-11 |
| 2 | Comando ausente o desconocido, nombre vacío, argumentos ausentes, sobrantes o no admitidos. | Error estándar con orientación de uso. | RF-1, RF-2, RF-6, RF-11 |

Mensajes propuestos: `Error: el hábito ya existe.`, `Error: el hábito no existe.`,
`Error: el nombre no puede estar vacío.`, `Error: no se pudieron leer los datos.`,
`Error: los datos guardados no son válidos.` y
`Error: no se pudieron guardar los cambios.`. Los errores esperables no muestran
trazas, estructuras internas ni mensajes técnicos sin traducir.

Orden de atención propuesto: capturar la fecha; interpretar argumentos y ayuda;
validar el nombre cuando corresponda; cargar y validar datos; aplicar la operación;
guardar solo si hay cambios; emitir el resultado. Por tanto, la ayuda no depende
del estado de los datos y un error sintáctico se informa antes de leerlos.

## 5. Decisiones técnicas justificadas y alternativas descartadas

**Cobertura: RF-1 a RF-11 y principios 1 a 6 de la constitución.**

| Decisión propuesta | Justificación | Alternativa descartada | RF |
| --- | --- | --- | --- |
| Python 3.12+ y biblioteca estándar; pytest solo en desarrollo. | Cumple la constitución y reduce el aprendizaje y mantenimiento. | Framework de CLI o dependencias de ejecución adicionales. | RF-1 a RF-11, soporte transversal |
| Interpretar comandos mediante `argparse`, con textos en español y abreviaciones de opciones desactivadas. | Valida argumentos sin dependencias; exige adaptar también encabezados y errores predeterminados. | Análisis manual de argumentos o bibliotecas externas. | RF-1, RF-6, RF-11 |
| Funciones puras en el núcleo y coordinación en la CLI. | Permite comprobar reglas sin archivos, consola ni reloj real. | Concentrar todas las responsabilidades en un único módulo. | RF-2 a RF-8 |
| Quitar espacios exteriores con `strip` y comparar mediante `casefold`. | Separa búsqueda sin mayúsculas de presentación; no elimina tildes ni espacios interiores. | Usar el texto literal como identificador o convertir permanentemente el nombre mostrado a minúsculas. | RF-2, RF-3, RF-4 |
| Orden técnico propuesto por el nombre comparado con `casefold`, sin configuración regional. | Es determinista entre equipos y sencillo de probar; el orden lingüístico de tildes y ñ sigue pendiente de aclaración. | Orden dependiente de la configuración regional del equipo. | RF-8 |
| Fechas con `datetime.date`, capturadas una sola vez por la CLI y recibidas explícitamente por el núcleo. | Evita cambios de referencia durante el comando y permite pruebas repetibles. | Consultar el reloj en cada regla o medir intervalos de 24 horas. | RF-4, RF-6, RF-7 |
| JSON local con nombres y fechas; rachas calculadas al consultar. | Cumple el almacenamiento requerido y evita rachas almacenadas que se queden obsoletas. | Base de datos, servicio remoto o guardar la racha como dato autoritativo. | RF-3 a RF-10 |
| Temporal en el mismo directorio y reemplazo con `os.replace`, tras sincronizar con `os.fsync`. | Mantiene el destino íntegro ante fallos de escritura previos al reemplazo. | Sobrescribir directamente el destino o crear el temporal en otro volumen. | RF-9, RF-10 |
| Resultados de dominio separados de mensajes de consola. | El núcleo no imprime; la CLI decide texto y código de salida. | Traducir cualquier excepción en éxito o mostrar trazas al usuario. | RF-3, RF-4, RF-5, RF-10, RF-11 |

Las reglas nuevas de producto no se consideran aprobadas por aparecer en este
plan. Antes de implementar el comportamiento afectado, quedan pendientes:

- **[NECESITA ACLARACIÓN] RF-2, RF-3 y RF-8:** admisión de caracteres de control,
  límites de longitud, equivalencia Unicode y orden lingüístico de tildes y ñ.
- **[NECESITA ACLARACIÓN] RF-9 y RF-10:** diferencia observable entre primer uso y
  desaparición de los datos de un usuario que ya había utilizado la aplicación.
- **[NECESITA ACLARACIÓN] RF-9 y RF-10:** uso simultáneo desde varios procesos. El
  reemplazo atómico evita archivos parciales, pero por sí solo no evita perder
  cambios de otro proceso; no se declara este escenario fuera de alcance.
- **[NECESITA ACLARACIÓN] RF-10:** garantías esperadas tras una interrupción abrupta
  o un corte de energía. La sincronización del contenido y el reemplazo no
  constituyen una garantía universal de durabilidad frente a cualquier fallo.
- **[NECESITA ACLARACIÓN] RNF-1:** criterio de revisión para considerar comprensibles
  la ayuda y los mensajes.

**Decisión aprobada en T02 — RF-6 y RF-7:** si el reloj retrocede o cambia la zona
horaria, las marcas que pasan a ser futuras se conservan y no se consideran
inválidas por ese motivo. No cuentan para la racha actual mientras sean futuras.
Se permite marcar la nueva fecha local de hoy: si ya tiene marca, se informa de
la repetición sin cambios; si no, se añade sin alterar las demás fechas. El núcleo
recibe la referencia explícita y el almacenamiento no rechaza una fecha válida
únicamente por ser posterior a ella. Los dos escenarios aprobados constan en los
casos límite de la especificación. No se necesita cambiar el esquema JSON.

La definición de datos inválidos de la sección 2 y los detalles observables del
contrato de la sección 4 son propuestas concretas de este plan. Deben quedar
reconciliados con la especificación antes de implementar cualquier comportamiento
que amplíe sus reglas. Este documento no modifica la especificación original.

## 6. Estrategia de tests

**Cobertura: RF-1 a RF-11; RNF-1 a RNF-5.**

Se utilizará pytest, sin dependencias adicionales de pruebas. Los casos se
identificarán por RF para mantener trazabilidad. Ninguna prueba accederá a datos
personales: la ruta de almacenamiento será temporal y controlada por la prueba.

| Grupo | Casos y comprobaciones | RF |
| --- | --- | --- |
| Nombres | Vacío, solo espacios, espacios exteriores, coincidencia entre «Leer» y « leer », conservación del nombre mostrado y búsqueda con otras mayúsculas. | RF-2, RF-3, RF-4 |
| Creación | Estado nuevo sin marcas, racha cero, duplicado rechazado y entradas originales sin modificación. | RF-3, RF-7 |
| Marcas | Hábito existente, inexistente, fecha recibida como hoy y repetición sin nuevo registro ni guardado. | RF-4, RF-5, RF-6 |
| Rachas | Sin registros; solo hoy; solo ayer; última marca anterior a ayer; secuencias que terminan hoy o ayer; huecos; entradas desordenadas y fechas repetidas en la entrada del algoritmo. | RF-5, RF-7 |
| Calendario | Cambio de mes, diciembre a enero, febrero bisiesto y no bisiesto, y límite inferior de fechas representables. | RF-6, RF-7 |
| Listado | Todos los hábitos una vez, nombre original, orden sin mayúsculas, valor de cada racha, listado vacío y ausencia de escrituras. | RF-7, RF-8 |
| Lectura y validación | Primer uso, recuperación del ejemplo, JSON vacío o mal formado, tipos incorrectos, versión desconocida, campos desconocidos, fechas inválidas y duplicados del modelo. | RF-2, RF-5, RF-9, RF-10 |
| Guardado y fallos | Ida y vuelta de datos; fallo al crear el temporal, escribir, sincronizar o reemplazar; destino anterior intacto y ausencia de confirmación de éxito. | RF-9, RF-10, RF-11 |
| Contrato CLI | Tres comandos, ayuda global y por comando, nombres con espacios, códigos 0/1/2, canales de salida, mensajes en español y ausencia de trazas esperables. | RF-1, RF-2, RF-3, RF-4, RF-5, RF-8, RF-11 |
| Entradas inválidas | Sin comando, comando desconocido, nombre ausente, nombre vacío, argumentos adicionales e intento de indicar otra fecha; datos sin cambios. | RF-1, RF-2, RF-6, RF-11 |
| Fecha capturada | Reloj controlado con una ejecución antes de medianoche y otra después; el reloj cambia durante una ejecución y la referencia inicial permanece fija. | RF-6, RF-7, RF-9 |
| Retroceso del reloj y zona horaria | Los dos escenarios de T02: conservación de marcas futuras, exclusión temporal de la racha y registro o repetición en la nueva fecha local, sin pérdida de datos. | RF-6, RF-7 |
| Aceptación entre ejecuciones | Crear, listar con cero, marcar, repetir la marca y volver a listar en nuevas ejecuciones; comprobar recuperación y efectos reales en el almacenamiento temporal. | RF-1 a RF-11 |

Los fallos de permisos y escritura se provocarán de forma controlada, sin
depender de los permisos particulares del equipo. Las pruebas de rechazo y
repetición compararán el contenido persistido antes y después. Las pruebas del
algoritmo podrán recibir fechas duplicadas para verificar que no se cuentan dos
veces, aunque el modelo persistido propuesto las rechace como inválidas.

Las pruebas completas comprobarán al menos el arranque real mediante
`python -m habits`, la conservación entre procesos y los códigos de salida.
Las pruebas dependientes de fechas usarán una fecha controlada internamente;
no se añadirá un comando ni una opción pública para alterar «hoy».

Las dudas abiertas no se convertirán en resultados esperados inventados. Sus
pruebas de aceptación se concretarán después de aclarar la especificación.

La verificación de cierre será `pytest -q`: cada regla deberá contar con pruebas
pertinentes y todas deberán pasar. También se revisarán manualmente la ayuda en
español, la trazabilidad de todos los RF y el respeto del alcance. Este plan no
afirma que las pruebas ni la aplicación estén ya implementadas.

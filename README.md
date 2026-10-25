# RachaEstudio

CLI educativa en Python para registrar hábitos de estudio y calcular rachas.
Pensada para estudiantes y para ser mantenida por un desarrollador junior.

## Datos de la tarea Planning

- **Nombre:** Luis Miguel Gonzales.
- **Proyecto:** habits-cli — RachaEstudio.
- **Repositorio:** [LuisMI2005-sudo/sddproyectoupb](https://github.com/LuisMI2005-sudo/sddproyectoupb).
- **Ficha para la entrega:** [ENTREGA.md](ENTREGA.md).
- **Tipo de entrega:** documentación de planificación de un proyecto propio.

## Presentación de la entrega

Este proyecto presenta un proceso de desarrollo dirigido por especificaciones
(SDD): primero se definen las reglas, después el comportamiento esperado, luego
el plan técnico y finalmente las tareas verificables.

El caso elegido es el seguimiento de hábitos de estudio mediante tres comandos:
crear un hábito, marcarlo como hecho hoy y listar sus rachas actuales. La propuesta
está personalizada para un estudiante que quiere mantener su constancia.

## Documentos y orden de lectura

1. [Constitución](docs/constitution.md): seis principios innegociables.
2. [Registro de aclaraciones](specs/001-habits-mvp/clarifications.md): decisiones
   aprobadas durante las preguntas iniciales y aclaraciones aún pendientes.
3. [Especificación del MVP](specs/001-habits-mvp/spec.md): usuarios, historias,
   once requisitos funcionales con criterios EARS, requisitos no funcionales,
   casos límite, exclusiones y criterios de finalización.
4. [Revisión QA de la especificación](specs/001-habits-mvp/qa-review.md):
   ambigüedades, contradicciones, casos no cubiertos y relación con la constitución.
5. [Plan técnico](specs/001-habits-mvp/plan.md): módulos, modelo JSON, algoritmo
   de racha, contrato de comandos, decisiones justificadas y estrategia de tests.
6. [Tareas](specs/001-habits-mvp/tasks.md): 37 tareas de hasta 30 minutos, con
   dependencias, RF relacionados y condiciones verificables de finalización.
7. [Revisión de cumplimiento](docs/revision-qa.md): estado real de la cobertura
   y de los criterios de finalización.

## Estado actual

La constitución, especificación, plan y desglose de tareas están redactados. T02
está completada como aclaración documental sobre cambios del reloj y zona horaria;
las demás tareas siguen pendientes.

La CLI y las pruebas todavía no están implementadas. Por tanto, esta entrega
documenta la especificación y planificación, y no presenta una aplicación
funcional ni una suite de pruebas aprobada. Las dudas de producto aún pendientes
están señaladas en el plan.

## Estructura

```text
RachaEstudio-SDD/
├── README.md
├── ENTREGA.md
├── AGENTS.md
├── .gitignore
├── docs/
│   ├── constitution.md
│   └── revision-qa.md
├── specs/
│   └── 001-habits-mvp/
│       ├── clarifications.md
│       ├── spec.md
│       ├── qa-review.md
│       ├── plan.md
│       └── tasks.md
├── habits/                 # Reservada para la implementación futura
└── tests/                  # Reservada para las pruebas futuras
```

Las carpetas reservadas contienen `.gitkeep` para conservar su estructura.

## Carpetas

- `docs/`: constitución y documentación del proyecto.
- `specs/`: especificaciones de funcionalidades.
- `habits/`: código del paquete Python.
- `tests/`: pruebas automatizadas.

Lee [la constitución](docs/constitution.md) y las instrucciones de `AGENTS.md`
antes de comenzar una implementación.

## Convenciones previstas

- Python 3.12 o superior; biblioteca estándar para la aplicación.
- Datos en JSON local, sin subir registros personales al repositorio.
- Identificadores en inglés y mensajes al usuario en español.
- Ejecución, cuando exista la CLI: `python -m habits <comando>`.
- Pruebas, cuando estén implementadas y `pytest` esté instalado: `pytest -q`.
- Antes de ejecutar pruebas en un entorno nuevo: `pip install pytest`.

El paquete de entrega excluye el entorno virtual, cachés, datos personales e
historial interno de Git. Para revisar la documentación no es necesario instalar
dependencias.

La presencia del enlace en este documento identifica el remoto configurado;
no acredita que los cambios locales ya hayan sido publicados en GitHub.

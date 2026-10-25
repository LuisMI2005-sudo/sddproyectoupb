# Revisión QA — Especificación del MVP de habits-cli

- **Documento revisado:** [spec.md](spec.md).
- **Referencia:** [constitución](../../docs/constitution.md).
- **Decisiones registradas:** [clarifications.md](clarifications.md).
- **Fecha:** 25 de octubre de 2026.
- **Alcance:** detección de ambigüedades, contradicciones, casos no cubiertos y
  conflictos con la constitución. No se proponen soluciones en esta revisión.

## 1. Ambigüedades restantes

- **QA-01 — RF-2 y RF-3:** no se definen los nombres con caracteres de control,
  tabulaciones o saltos de línea, ni un límite de longitud. Pendiente en T01.
- **QA-02 — RF-2:** no se precisa si dos representaciones Unicode visualmente
  equivalentes identifican el mismo hábito. Pendiente en T01.
- **QA-03 — RF-8:** no se concreta el orden de tildes, ñ, números y símbolos.
  El orden técnico del plan es una propuesta, no una nueva aprobación. T01.
- **QA-04 — RF-1 y RF-11:** la spec no concreta la invocación de ayuda, el resultado
  al ejecutar sin comando ni los códigos de salida. El plan propone un contrato
  que debe reconciliarse con la spec en T07.
- **QA-05 — RF-10:** «datos dañados» no delimita por sí solo los casos de tipos,
  fechas, nombres y duplicados inválidos. El plan propone validaciones; T07 debe
  reconciliarlas con los criterios funcionales.
- **QA-06 — RNF-1:** claridad y brevedad no tienen una condición de aceptación
  acordada. Pendiente en T06.

## 2. Contradicciones entre requisitos

No se detectaron contradicciones directas entre RF-1 y RF-11. Conservar el nombre
de presentación es compatible con comparar sin mayúsculas. Conservar la racha de
ayer es compatible con registrar solo hoy.

La afirmación inicial de que no quedaban dudas abiertas era inconsistente con los
hallazgos del QA. Esa afirmación se corrigió al registrar T02: la spec ahora
reconoce las aclaraciones pendientes.

## 3. Casos límite no cubiertos completamente

- **QA-07 — RF-9 y RF-10:** desaparición de datos después de un uso previo, frente
  al inicio legítimo sin datos. Pendiente en T03.
- **QA-08 — RF-9, RF-10 y RNF-3:** dos procesos guardando cambios simultáneamente;
  no está aprobado el resultado ni una exclusión de ese escenario. T04.
- **QA-09 — RF-10:** interrupción abrupta durante el guardado y garantías al
  reiniciar, diferenciadas de un corte de energía. Pendiente en T05.
- **QA-10 — RF-1 y RF-10:** consulta de ayuda con datos dañados. El plan propone
  independencia de la ayuda, pendiente de reconciliación documental en T07.
- **QA-11 — RF-11:** concurrencia de un error de argumentos y datos dañados; la
  prioridad aparece propuesta en el plan, pendiente de reconciliación en T07.

## 4. Conflictos con la constitución

No se detectaron conflictos explícitos. Hay una brecha de verificabilidad entre
la validación exigida por el principio 5 y la definición incompleta de datos
dañados de RF-10, indicada en QA-05.

La ausencia de detalles de stack o módulos en la spec funcional no contradice la
constitución: el plan trata esas decisiones técnicas. Sin implementación todavía
no puede verificarse el cumplimiento material de esos principios.

## 5. Hallazgo resuelto documentalmente

- **QA-12 — RF-6 y RF-7:** los retrocesos del reloj y cambios de zona horaria
  estaban indeterminados. El usuario aprobó los dos escenarios de T02 y sus
  reglas se incorporaron a RF-6, RF-7 y los casos límite. Véase D-07 en el registro
  de aclaraciones. Resuelto documentalmente; no implementado ni probado.

## 6. Cobertura ejecutable y veredicto

La auditoría de tests y criterios de finalización se conserva en
[revision-qa.md](../../docs/revision-qa.md). No existen todavía tests ni código de
la aplicación. La planificación está documentada, pero no acredita el cumplimiento
de la spec por una aplicación funcional.

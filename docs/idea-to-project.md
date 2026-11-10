# Recorrido mínimo de Blends: de idea a proyecto

Fecha: 2026-10-30. Estado: propuesta de proceso, basada en el ciclo vigente del sistema existente. Es diseño documental, no un flujo implementado ni autorización para operar el portfolio actual.

## Alcance y responsabilidades

Blends coordina el tramo de idea: el [ingreso previo](idea-intake.md), la exploración, definición y solicitud de decisión, y la preparación del contexto de proyecto. Después de graduar, Cellar conduce el proyecto bajo el método de Blends, según la [frontera](https://github.com/denispaiva/vcx/blob/main/docs/architecture/responsibility-matrix.md#1-método-y-conducción) derivada de VCX-D029. Cellar conserva la información según su [modelo mínimo](https://github.com/denispaiva/cellar/blob/main/docs/initiative-model.md). El ingreso utiliza la capacidad requerida de Brix para preparar el documento y sus preguntas asesinas; la exploración posterior también puede producir evidencia manual. Esa capacidad de ingreso aún no está implementada en VCX.

El responsable del portfolio conserva la decisión de graduar, diferir o archivar, y asigna el PO del proyecto en esa decisión. No se crean código, entornos, despliegues o gastos como efecto implícito de una graduación. Las reglas generales pertenecen a Blends; los parámetros y aprobaciones particulares, al Cellar afectado.

## Secuencia

| Paso | Blends coordina | Cellar conserva | Brix puede aportar | Condición de salida |
|---|---|---|---|---|
| 1 · Registro después del ingreso | Comprobar la confirmación humana posterior al reporte del consejo y registrar la propuesta aceptada | Identidad, captura original, documento enriquecido, reporte y decisión; estado `registered` | Preparación previa mediante `/idea` e instrumento de preguntas asesinas | Se cumplieron las dos revisiones humanas del ingreso: OK del documento antes del consejo y confirmación posterior al reporte |
| 1.1 · Arquitectura de referencia y FinOps | Coordinar modelo propuesto, aplicación de plantilla FinOps y revisión de presupuesto | Modelo, revisión, escenarios, fuentes, estimaciones y decisión del alcance autorizado; la idea sigue `registered` | Capacidad de arquitectura de referencia y cálculos o validaciones disponibles | El humano dispone de costos trazables al modelo y decide el presupuesto de la fase solicitada |
| 2 · Exploración | Acotar preguntas y solicitar presupuesto de exploración al responsable del portfolio | Presupuesto autorizado, preguntas, evidencia y estado `exploring` | Cálculos o evaluaciones con permisos explícitos, si se necesitan | Cada pregunta decisiva tiene respuesta y evidencia; no se amplía silenciosamente el alcance |
| 3 · Definición | Preparar propuesta de valor, vía de validación propuesta, con MVP recomendado y no obligatorio, y alternativas | Definición y solicitud de gate en `gate-ready` | Instrumentos de análisis cuando aporten valor | La autoridad recibe un caso decidible y sus riesgos; no un compromiso de construcción |
| 4 · Gate | Comprobar criterios, presentar recomendación y pedir decisión | Decisión explícita sobre una revisión identificada | Evidencia o recomendación, sin autoridad para aprobar | La autoridad competente elige graduar, diferir o archivar, o la solicitud queda pendiente |
| 5 · Preparación | Aplicar una graduación autorizada y preparar contexto de gobierno | Contexto de proyecto con PO asignado, historia enlazada y única fuente activa de seguimiento; desde aquí Cellar conduce | Generación de documentos, solo si existe una capacidad apta | El contexto queda verificable antes de publicar `kind=project` y `lifecycleState=graduated`; el traspaso concreto a Cellar sigue pendiente de diseño |

Antes del paso 1, el [ingreso](idea-intake.md) conserva una propuesta en preparación, fuera del índice de ideas aceptadas. La captura no incorpora código, arquitectura completa o dependencias de otros productos. Misión y visión pueden formularse como propuestas al nutrir el documento; se refinan durante la definición y forman el contexto explícito del proyecto.

El paso 1.1 se detalla en [arquitectura de referencia y FinOps](architecture-and-finops.md). Se propone un modelo orientativo para estimar escenarios, no una arquitectura final de implementación. La exploración comienza con el presupuesto de su fase autorizado; proyecciones de construcción u operación no autorizan esas fases.

El reporte inicial del consejo no obliga a repetir la misma consulta durante la exploración. Una nueva corrida debe justificar preguntas o material nuevo y respetar los permisos, presupuesto y trazabilidad del método adoptado.

## Gate y alternativas

El gate ordinario requiere preguntas decisivas respondidas con evidencia, ausencia de un bloqueo para operar y capacidad real del portfolio. La exploración necesita un presupuesto declarado por el responsable del portfolio; el ejecutor no lo inventa.

La historia existente incluye graduaciones por desviación deliberada de criterios de evidencia. El diseño las admite solo cuando la autoridad competente registra explícitamente qué exceptúa, por qué, con qué riesgos y sobre qué revisión. No convierte una excepción en cumplimiento ni permite anular obligaciones fuera de su autoridad.

- **Graduar:** habilita preparar gobierno y contexto del proyecto. No autoriza el primer plan de construcción por sí sola.
- **Diferir:** conserva el contexto y exige un disparador explícito de revisión. No equivale a crear un proyecto parado.
- **Archivar:** conserva captura, evidencia, razón y condición de posible revisión. No borra la iniciativa.
- **Sin respuesta:** mantiene el gate pendiente; una recomendación o el paso del tiempo no lo aprueban.

## Resultado mínimo de la preparación

Identidad estable; misión y visión; alcance; autoridad y perfil adoptados; referencia canónica a planificación y continuidad; acta de graduación; referencias a la captura y la evidencia; y conformidad con las convenciones aplicables. Lo que todavía no exista se declara pendiente, sin inventar código, versiones o evidencia.

Blends debe permitir cargar ese contexto sin cargar todo el portfolio. La documentación de exploración queda preservada y apunta al contexto vigente; el índice del Cellar es una localización, no otro roadmap editable.

## Cambios e interrupciones

Una solicitud de decisión identifica la revisión evaluada. Cambiar materialmente propósito, evidencia o riesgos después de aprobar exige una nueva revisión de la decisión antes de aplicar la graduación. Cambiar un enlace sin alterar el contenido revisado debe conservar procedencia verificable; no se considera aprobación de otra revisión por inferencia.

La aplicación se identifica de forma única respecto de la decisión y debe ser reanudable: reintentar la misma graduación no crea otro proyecto o acta. Antes de publicar la transición se comprueban documentos y referencias. Si la preparación se interrumpe, se conserva la decisión y se informa «graduación aprobada, preparación pendiente», con el ciclo aún en `gate-ready`; no se informa éxito ni se genera otra fuente activa. La implementación futura deberá ofrecer publicación consistente o recuperación verificable según el almacenamiento elegido.

Una decisión sobre otro Cellar, de alguien sin autoridad o sobre una revisión no vigente se rechaza sin alterar el estado publicado. Un documento faltante mantiene pendiente la preparación. Se conserva evidencia del intento y se permite corregir la causa sin reescribir el acta original.

## Siguiente límite

El primer plan de trabajo, el código, los entornos, la entrega y las métricas de operación pertenecen a la conducción del proyecto en Cellar, bajo el método de Blends; ese recorrido y el traspaso desde la preparación están pendientes de diseño. El estado del ciclo de idea y el estado de trabajo siguen siendo dimensiones separadas.

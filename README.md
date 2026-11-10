# Blends

Proyecto de primer nivel de VCX, par de Cellar. Primera curación documental: 2026-10-30, bajo el acuerdo VCX-D020 del [registro](https://github.com/denispaiva/vcx/blob/main/docs/decisions/README.md).

## Misión

Definir el método con el que personas, equipos y agentes convierten ideas en resultados verificables, y coordinar las actividades e hitos de cada idea hasta su graduación, mediante una experiencia integrada de planificación, coordinación y gobierno.

## Visión

Que distintos equipos puedan adoptar un proceso explícito y configurable para sus portfolios, con decisiones trazables y continuidad entre sesiones, sin depender de los nombres o la disposición del portfolio original.

## Destinatarios y alcance

Se dirige a responsables de portfolios, equipos de proyecto y agentes autorizados. Es dueño del método: etapas, compuertas, criterios de evidencia, roles y plantillas de proceso. Coordina ingreso, exploración, definición, gate y preparación del contexto de proyecto, y la presentación del portfolio. No conduce proyectos: Cellar los conduce bajo ese método, según la [frontera](https://github.com/denispaiva/vcx/blob/main/docs/architecture/responsibility-matrix.md#1-método-y-conducción) derivada de VCX-D029.

**Entradas previstas:** configuración de un Cellar, políticas y versiones adoptadas, estado de las iniciativas, solicitudes de trabajo y permisos de sus participantes.

**Resultados previstos:** operaciones coordinadas, seguimiento y vistas del portfolio, comprobaciones de proceso y solicitudes de decisiones con evidencia.

Blends consume las interfaces públicas de Cellar para estado y de Brix para capacidades. No almacena otra copia editable del portfolio dentro de su implementación ni decide por sí mismo presupuestos o excepciones reservadas a una autoridad humana.

## Estado y contexto

El [recorrido de idea a proyecto](docs/idea-to-project.md) precisa el proceso del [primer corte conjunto con Cellar](https://github.com/denispaiva/vcx/blob/main/docs/superpowers/specs/2026-10-30-cellar-blends-first-journey-design.md). Se documenta como propuesta, sin convertirlo en un flujo implementado.

La [entrada de propuestas](docs/idea-intake.md) incorpora la preparación en Brix, preguntas asesinas, revisión humana, consejo por archivo y reporte. Solo la confirmación humana posterior al reporte habilita registrar la idea.

Después del ingreso, la [etapa de arquitectura de referencia y FinOps](docs/architecture-and-finops.md) prepara el modelo y los escenarios para decidir el presupuesto de exploración, sin autorizar por ello construcción u operación.

Desde el 2026-11-10 este repositorio trae también los **estándares del método**, curados desde el precursor (ver [Origen](#origen)). La maquinaria con la que el precursor aplicaba el método (validadores, `verify`, tren, cola y tablero) no viajó: no tenía consumidores fuera de él, y queda en el precursor archivado (VCX-D063).

Para trabajar: leer este documento y [AGENTS.md](AGENTS.md). Consultar después la [matriz](https://github.com/denispaiva/vcx/blob/main/docs/architecture/responsibility-matrix.md), [interoperabilidad](https://github.com/denispaiva/vcx/blob/main/docs/architecture/interoperability.md) o [decisiones](https://github.com/denispaiva/vcx/blob/main/docs/decisions/README.md), según la tarea. La [procedencia histórica](https://github.com/denispaiva/vcx/blob/main/docs/sources.md) es referencia, no un requisito de instalación.

## Estándares del método

| Estándar | De qué trata |
|---|---|
| [ci-cd](standards/ci-cd.md) | Cómo se verifica, se integra y se despliega el trabajo |
| [conventions](standards/conventions.md) | Nombres, escritura, commits, ramas y entornos |
| [idea-lifecycle](standards/idea-lifecycle.md) | Ciclo de vida de una idea hasta su graduación |
| [role-matrix](standards/role-matrix.md) | Roles, planificación proporcional y gobierno, con su [esquema](standards/role-matrix.schema.json) y el del [plan de trabajo](standards/work-plan.schema.json), más ejemplos |
| [security](standards/security.md) y [secrets](standards/secrets.md) | Protección de datos, superficies y secretos |
| [skills](standards/skills.md) | Cómo entra una skill a la biblioteca |
| [work-proportionality](standards/work-proportionality.md) | Cuánto rigor corresponde a cada cambio |

Los estándares nombran proyectos y casos del portfolio donde nacieron, como ejemplos. Los inventarios, importes y datos de infraestructura de esa instancia quedaron fuera.

## Fuentes de trabajo y dependencias

| Tema | Fuente canónica y situación |
|---|---|
| Preparación y admisión | [Ingreso de ideas](docs/idea-intake.md): documento, OK de envío, consejo, reporte y confirmación de ingreso; diseño sin comandos nuevos implementados |
| Exploración y graduación | [Recorrido mínimo](docs/idea-to-project.md), propuesto; conserva decisiones humanas y preparación reanudable |
| Arquitectura y presupuesto | [Etapa FinOps](docs/architecture-and-finops.md), diseño requerido; estimar no autoriza gasto |
| Información que consume | [Cellar](https://github.com/denispaiva/cellar/blob/main/README.md) y su modelo; contratos todavía propuestos |
| Instrumentos que invoca | [Brix](https://github.com/denispaiva/brix/blob/main/README.md); interfaces objetivo y compatibilidad pendientes de implementación |
| Autoridad e interfaces compartidas | [Decisiones](https://github.com/denispaiva/vcx/blob/main/docs/decisions/README.md), [interoperabilidad](https://github.com/denispaiva/vcx/blob/main/docs/architecture/interoperability.md) y [matriz](https://github.com/denispaiva/vcx/blob/main/docs/architecture/responsibility-matrix.md) |
| Trabajo documental de esta etapa | [Plan de curación](https://github.com/denispaiva/vcx/blob/main/docs/superpowers/plans/2026-10-30-blends-cellar-curation.md) y [reporte de ejecución](https://github.com/denispaiva/vcx/blob/main/docs/superpowers/reports/2026-10-30-blends-cellar-curation.md) |

El seguimiento operativo existente permanece en el repositorio de Blends: `vc/framework/.roadmap.json` registra Blends Kernel y la rama `queue` conserva el backlog. Estas son localizaciones de origen, no nuevas dependencias de instalación de VCX. La [correspondencia fechada](https://github.com/denispaiva/vcx/blob/main/docs/architecture/current-system-mapping.md) identifica sus fuentes y límites; no es un monitor actualizado ni autoriza su traslado. La [procedencia](https://github.com/denispaiva/vcx/blob/main/docs/sources.md) conserva las rutas históricas.

Cellar custodia la información y conduce los proyectos graduados; Brix aporta instrumentos; Blends define el método y coordina el tramo de idea. Las decisiones de ingreso, presupuesto, excepciones y graduación corresponden al responsable del portfolio; las del proyecto, a su PO. La curación no asigna roles nuevos ni convierte el Kernel existente en un producto distinto.

## Próximo entregable propuesto

Precisar una operación documental de ingreso y registro sobre el modelo de Cellar: entradas, salidas, revisión evaluada, permisos, confirmaciones humanas y recuperación ante interrupciones. Se considerará aceptable si el caso sintético distingue el OK para enviar al consejo de la confirmación posterior para ingresar, conserva las fuentes y revisiones y explica cómo reanudar sin registrar dos iniciativas.

Ese entregable requiere un alcance posterior y no implementa `/idea`, `/council` o una API. Antes de una adaptación ejecutable habrá que resolver la compatibilidad de Brix y la conciliación con el Kernel aprobado, manteniendo sus decisiones en el registro compartido. La pertenencia del seguimiento de este producto a un Cellar real continúa pendiente.

## Origen

Este repositorio nació el 2026-11-10 con **historia nueva** (VCX-D032, opción B'). El historial de los estándares en el precursor contenía identificadores de infraestructura e importes de una instancia privada, y filtrarlo por ruta los habría publicado. La procedencia es `denispaiva/blends-precursor` (privado y archivado) en `51fbd2a1`, para `standards/`, y `denispaiva/vcx` en `blends/`, para el resto. Los enlaces a archivos que no viajaron quedaron como texto con su ruta de origen.


## Licencia

MIT. Ver [LICENSE](LICENSE).

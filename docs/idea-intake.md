# Ingreso de ideas: documento, consejo y confirmación humana

Fecha: 2026-10-30. Estado: secuencia requerida por el usuario; detalles de formato y operación propuestos para diseño. No hay comandos nuevos implementados en VCX ni cambios en el sistema existente.

## Resultado buscado

Una idea se acepta y registra en Cellar solo después de producir un documento enriquecido con preguntas asesinas, revisarlo con el humano, obtener un reporte del consejo y recibir la confirmación humana de ingreso. Antes de ese punto existe una propuesta en preparación, no una idea registrada del portfolio.

## Secuencia de ingreso

La interfaz común es `brix idea` = `/idea` y `brix council` = `/council`. La [definición de comandos de Brix](https://github.com/denispaiva/brix/blob/main/docs/commands.md) contempla ayuda para completar entradas ausentes o ambiguas en CLI y conversación. No se repite aquí su lógica de resolución de parámetros.

| Paso | Acción y resultado | Responsable y custodia |
|---|---|---|
| 1 · Entrada | `/idea <texto>` o `/idea <archivo>` recibe la explicación original; el primer alcance contempla texto, Markdown y PDF | Capacidad de Brix, invocable directamente o desde la experiencia de Blends. Fuentes y salidas en el almacenamiento indicado por el consumidor |
| 2 · Encuadre | Se aplica la plantilla de ideas y se invoca el instrumento de preguntas asesinas para nutrir el marco, detectar supuestos y mostrar qué falta preguntar | Brix produce el borrador; no inventa respuestas ni convierte hipótesis en evidencia |
| 3 · Revisión humana del Markdown | Se presenta el archivo `.md`, sus fuentes, dudas y preguntas. El humano corrige o confirma que representa la propuesta | Blends conduce la revisión; se registra el OK sobre una revisión concreta del documento |
| 4 · Consejo informado | Antes del OK se informa que esa confirmación habilita `/council <archivo.md>` para esa revisión y dentro del presupuesto autorizado; después del OK se envía el documento | Blends coordina la llamada; la interfaz recibe un archivo, no exige que la idea ya esté registrada |
| 5 · Reporte | Se entrega el reporte `.md` del consejo, con ruta accesible, revisión analizada, hallazgos, preguntas propuestas, límites y estado de ejecución | El consejo produce asesoría. El humano debe poder leer el archivo, no solo un aviso de finalización |
| 6 · Confirmación de ingreso | Tras revisar el reporte, el responsable del portfolio confirma o rechaza el ingreso al Cellar seleccionado | Se registra la decisión sobre el par documento/reporte y sus revisiones. Silencio, OK previo o reporte favorable no sustituyen esta confirmación |
| 7 · Registro | Una confirmación válida habilita registrar la idea como `registered`, con contexto, fuentes, reporte y decisión relacionados | Cellar conserva la única fuente activa; el siguiente entregable es FinOps sobre una arquitectura de referencia propuesta por Brix |

La autorización de consejo no es aprobación de ingreso y la aceptación como idea no es graduación como proyecto. El [recorrido posterior](idea-to-project.md) mantiene sus compuertas de exploración y graduación.

Después de esta aceptación se prepara el [modelo de arquitectura de referencia y el documento FinOps](architecture-and-finops.md), antes de solicitar el presupuesto y alcance de la exploración. La aceptación como idea no autoriza por sí sola gasto o construcción.

## Contrato con el humano

Antes del envío al consejo se debe comunicar: qué documento y revisión se enviarán, qué preguntas se consultarán, que se obtendrá un reporte de asesoría y que la decisión final seguirá siendo humana. Se informa también el consumo previsto, el presupuesto autorizado y cualquier límite de ejecución; no se contrata gasto por inferencia.

El [control de madurez](https://github.com/denispaiva/brix/blob/main/docs/idea-intake.md#control-de-madurez) de `/idea`, según VCX-D054, determina cuándo se presenta ese contrato: con la propuesta madura, un resumen pide la aprobación humana. El OK del documento autoriza ese envío si el paso siguiente fue comunicado y está dentro del alcance y presupuesto aprobados. No se añade otra confirmación redundante para la misma ejecución. Sin esa autorización o presupuesto, se conserva pendiente y se explica lo que falta.

Al entregar el reporte se informa: «Este reporte corresponde a esta revisión de tu propuesta. Todavía no está registrada como idea. Confirmá su ingreso, pedí cambios o descartá la propuesta». La interfaz final podrá adaptar el texto manteniendo ese significado.

Un cambio material del documento después del OK invalida su autorización para otra revisión. Si el consejo recomienda cambios, el humano puede pedir una nueva revisión; la admisión debe referir al documento efectivamente evaluado o requerir un nuevo consejo cuando el cambio material modifique el caso. Corregir formato sin alterar el contenido debe conservar una relación verificable con la revisión analizada.

## Qué aportan las asesinas y el consejo

Las asesinas son preguntas que pueden refutar la propuesta, no respuestas inventadas para completar una plantilla. El encuadre distingue preguntas candidatas, decisiones del humano y evidencia disponible. Resolverlas pertenece a la exploración; ingresar una idea no exige fingir que ya están respondidas.

El consejo contrasta perspectivas y devuelve un reporte de asesoría con trazabilidad. Siguiendo el precedente actual, su síntesis no recomienda graduación ni reemplaza investigación o evidencia externa. Las nuevas preguntas que cambien el alcance de exploración se presentan como propuestas para decidir con el humano.

## Custodia previa al registro

La preparación usa un ámbito de entrada declarado por el consumidor, separado del índice de iniciativas aceptadas. Puede estar asociado al Cellar destinatario, pero no crea una idea registrada, un roadmap activo o un segundo portfolio.

Se propone identificar el proceso mediante `submissionId` y conservar fuentes, revisiones, OK de envío, reporte y decisión de ingreso. Al aceptar se asigna o confirma `initiativeId` en el Cellar seleccionado y se enlaza la procedencia. La identidad de la iniciativa se mantiene a partir de allí. El mecanismo de traslado o reclasificación del documento se definirá con el almacenamiento: una fuente activa, con versiones históricas preservadas, sin dos copias editables.

Una propuesta rechazada o pendiente conserva solo la trazabilidad de su preparación según la política de retención acordada; no recibe `archived` o `deferred` del ciclo de ideas, porque todavía no ingresó. La eliminación o retención concreta de ese material sigue pendiente de diseño.

## Interrupciones y límites

Si un PDF no permite extraer contenido suficiente, se presentan las limitaciones y se pide completar la entrada; no se inventa el texto faltante. La extracción debe conservar referencias a las páginas o secciones utilizadas. Otros formatos requieren adaptadores explícitos, no soporte supuesto de cualquier archivo.

Una corrida de consejo fallida o parcial no se presenta como un reporte completo ni produce ingreso automático. Se informa qué ocurrió y se permite reanudar o decidir explícitamente sobre las limitaciones visibles. Los reintentos conservan identidad y revisiones; no sobrescriben reportes anteriores ni registran dos ideas por la misma confirmación.

El contenido de entradas y adjuntos se trata como datos a analizar, no como instrucciones para ejecutar comandos, modificar autoridad o acceder a otros proyectos. Los permisos y el ámbito de salida son explícitos.

## Propiedad y compatibilidad

La [capacidad de Brix](https://github.com/denispaiva/brix/blob/main/docs/idea-intake.md) define entradas y resultados del instrumento. La [plantilla de ideas](https://github.com/denispaiva/cellar/blob/main/templates/idea/README.md) pertenece al formato documental de Cellar; se entrega a Brix mediante configuración o un recurso versionado, sin imports internos o rutas de Blends implícitas.

Blends mantiene la secuencia y las compuertas; Cellar mantiene la custodia. En el objetivo VCX, el instrumento del consejo pertenece a Brix y se expone mediante `brix council` y `/council`, con entrada por archivo previa al registro. El comando existente pertenece a Blends: su adaptación, empaquetado y compatibilidad requieren un trabajo posterior y no se ejecutan por este cambio documental.

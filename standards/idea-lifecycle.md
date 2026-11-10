# Estándar: ciclo de vida de una idea

> Cómo entra una idea al portfolio, qué tiene que responder para seguir viva, y
> quién decide si gradúa, espera o se archiva.
> Versión 1.0.0 — 2026-08-23.

## Objetivo

`ideas/` existía antes que este documento y ya tenía una convención de entrada:
el wizard `./idea` (precursor, `ideas/README.md` en `51fbd2a1`) crea la ficha, el archivo de
seguimiento y la entrada del tablero. Lo que no estaba escrito era lo que pasa
**después** de la captura: cuándo una idea se considera explorada, qué habilita
la decisión de invertir, y qué hace el portfolio con una idea que dejó de
moverse.

Este estándar formaliza esa secuencia. No inventa un proceso ajeno: describe lo
que el portfolio ya practicó dos veces —
Teyst (precursor, `vc/teyst/docs/decisions/0001-graduation.md` en `51fbd2a1`) y
Pinot Index (precursor, `vc/pinot-index/docs/decisions/0001-graduation.md` en `51fbd2a1`), ambas
graduadas el 2026-08-13 — con los nombres, los estados y las reglas duras que
esas dos graduaciones dejaron implícitos.

Lo que este documento **no** cambia: la forma de crear una idea (sigue siendo el
wizard), la separación de idiomas de `AGENTS.md` (precursor, `AGENTS.md` en `51fbd2a1`), ni
ninguna invariante de [`role-matrix.md`](role-matrix.md). Extiende; no
reemplaza.

## Vocabulario

| Concepto | Significado |
|---|---|
| **Pregunta abierta** | Algo que la idea todavía no sabe. Se registra como pregunta, nunca como supuesto disfrazado de decisión. |
| **Pregunta asesina** | Pregunta abierta cuya respuesta puede **matar** la idea. Se nombra en la captura y acota el alcance de la exploración. |
| **Presupuesto de exploración** | Timebox o tope de esfuerzo declarado antes de empezar a explorar. |
| **Gate de graduación** | Punto de decisión del PO: graduada, diferida o archivada. |
| **Scaffolding de gobernanza** | Roadmap, decisiones, contexto de agentes y tablero de un proyecto nuevo. Nunca incluye código. |

Los estados se escriben en inglés dentro del archivo de seguimiento y se
presentan en español en la ficha y el tablero, según la convención de idioma del
workspace.

| Estado interno | Presentación | Paso |
|---|---|---|
| `registered` | Registrada | 1 — Captura |
| `exploring` | En exploración | 2 — Exploración |
| `gate-ready` | Lista para gate | 3 — Definición |
| `graduated` | Graduada | 4/5 — Gate y graduación ejecutada |
| `deferred` | Diferida | 4 — Gate |
| `archived` | Archivada | 4 — Gate |

## Paso 1 — Captura

**Estado resultante: `registered` (Registrada).**

Entra una idea al pipeline. La salida es un documento de idea, y nada más.

Qué contiene el documento:

- Las palabras del PO, capturadas fielmente. La captura transcribe; no mejora,
  no completa, no traduce a jerga de producto.
- El modelo explícito: quién está de cada lado, quién paga a quién, qué tiene
  que ser cierto para que exista un producto. Si el PO lo dejó implícito, la
  captura lo explicita y lo marca como explicitación, no como cita.
- Las preguntas abiertas, **como preguntas**. Una pregunta contestada de oficio
  para que el documento se vea más completo es peor que la pregunta.
- Entre ellas, las **preguntas asesinas** nombradas como tales.

La rama de trabajo de una idea usa el prefijo `idea/<slug>` — `idea/terroir`,
nunca `feat/terroir`. Los prefijos `feat/`, `fix/`, `chore/` y `experiment/`
pertenecen a proyectos ya graduados a `vc/`; ver
[convenciones](conventions.md#ramas-y-commits). El nombre de la rama es la
primera afirmación pública sobre en qué etapa está una idea, y `feat/` afirma un
compromiso de construcción que el pipeline todavía no tomó.

Reglas duras de este paso:

- **Cero código.** Ninguna línea, en ningún lenguaje, en ningún lado.
- **Cero estructura de `vc/`.** Ni carpeta, ni matriz de gobernanza, ni work
  plan.
- **Cero referencias a otros proyectos del portfolio.** Una idea nace sin
  acoplamientos. Si al graduar aparece conocimiento reutilizable, ese es el
  momento de decidir dónde vive — no antes.

**Por qué las asesinas se nombran acá y no en el paso 2:** sin ellas, la
exploración no tiene alcance cerrado y se expande hasta consumir la atención
disponible. Nombrarlas en la captura convierte el paso siguiente en una lista
finita de preguntas con respuesta pendiente, en vez de una investigación
indefinida.

Una idea puede permanecer `registered` sin límite. Es el único estado sin
vencimiento: el portfolio puede sostener ideas capturadas que nadie está mirando
— eso no es deuda, es inventario.

## Paso 2 — Exploración

**Estado resultante: `exploring` (En exploración).**

Se responden **solo las preguntas asesinas**. Las demás siguen abiertas y no
bloquean nada: si una no-asesina impide decidir, es que estaba mal clasificada,
y reclasificarla es un cambio registrado en el documento, no un ensanche
silencioso del alcance.

Cada respuesta llega **con evidencia**: investigación con fuente citada,
conversaciones reales con personas reales, números verificados contra su origen.
Opinión fundada no es evidencia; una respuesta sin evidencia deja la pregunta
abierta.

**Campo obligatorio al entrar: el presupuesto de exploración.** Un timebox
(fechas) o un tope de esfuerzo declarado. **Sin tope declarado no se entra al
paso 2.** Es la regla FinOps del framework — ningún consumo empieza sin budget
(invariante 22, «Allocation completa», de
[`role-matrix.md`](role-matrix.md)) — aplicada al recurso que la etapa de
idea realmente gasta, que es atención, no dinero.

El presupuesto lo declara el PO. Un ejecutor no lo estima por su cuenta: sería
decidir cuánta atención del portfolio se compromete, que es una decisión de
inversión.

## Paso 3 — Definición

**Estado resultante: `gate-ready` (Lista para gate).**

Con las asesinas respondidas, la idea se convierte en algo decidible:

- **Propuesta de valor concreta**: para quién, qué problema, por qué esto y no
  lo que ya existe.
- **Hipótesis de MVP**: la versión más chica que probaría o refutaría la
  propuesta.
- **Modelo elegido** entre las alternativas que la exploración dejó vivas, con
  el motivo de la elección y las alternativas descartadas registradas.

El entregable es un **PRD-lite**: suficiente para decidir una inversión, no para
construir. La distinción es la que separa este paso del trabajo del proyecto
graduado: acá se responde "¿vale la pena?", no "¿cómo se hace?".

**En brownfield este paso es el assessment.** Precedente real: la Fase 1 de
Teyst — contrato de frontera, inventario de reutilización, inventario del
entorno y verificación del hallazgo de seguridad — vivió entera en
`ideas/teyst/assessment/` y fue lo que el PO revisó antes de graduar. Un
proyecto que ya existe llega al gate con inventario, no con hipótesis.

## Paso 4 — Gate de graduación

**Decisión del PO.** No es un paso de trabajo: es un punto de decisión, y la
[invariante 44](role-matrix.md) aplica en su forma más literal — la
recomendación del ejecutor no constituye autorización, y el gate permanece
abierto hasta respuesta explícita.

La decisión se registra como **registro formal de decisión**, siguiendo el
precedente de los dos `0001-graduation.md` existentes: fecha, autoridad, estado,
motivo, evidencia, riesgo abierto que sobrevive a la decisión, y una sección
explícita de **qué NO decide** este registro.

Criterios del gate:

1. **Asesinas respondidas con evidencia.** Todas. Una sola sin responder deja la
   idea en el paso 2.
2. **Sin bloqueo regulatorio.** No "sin riesgo regulatorio" — sin *bloqueo*: un
   riesgo conocido y acotado se hereda como riesgo abierto, igual que hicieron
   las dos graduaciones existentes; un impedimento legal para operar no.
3. **Capacidad real del portfolio.** Si nadie puede tomarla ahora, la respuesta
   honesta es diferirla, no graduarla y dejarla parada ocupando gobernanza.

Tres veredictos posibles:

- **Graduada.** Nace `vc/<nombre>/`. Si el proyecto va a gastar, nace también su
  línea en el ledger FinOps — no después.
- **Diferida.** Con **disparador explícito**: el evento concreto que la vuelve a
  poner sobre la mesa. "Algún día", "cuando haya tiempo" o "más adelante" no son
  disparadores; una idea diferida sin disparador está archivada y conviene
  decirlo.
- **Archivada.** La idea no sigue.

**Archivar es un éxito del pipeline, no un fracaso.** Una idea matada en el gate
costó documentos y algunas conversaciones. La misma idea matada seis meses
después de graduar cuesta repos, entornos, gobernanza instanciada, líneas de
billing y el trabajo de desarmar todo eso. El pipeline existe para que el
descarte pase temprano y barato.

## Paso 5 — Graduación ejecutada

Lo que se instancia al graduar es **scaffolding de gobernanza**: archivo de
seguimiento en `vc/<nombre>/`, registro de decisión, matriz de autoridad,
contexto de agentes y entrada en el tablero. El archivo de seguimiento de la
idea se muda a `vc/<nombre>/` — es lo que hicieron las dos graduaciones
existentes, y por eso una idea graduada deja de aparecer como idea en el
tablero.

La carpeta `ideas/<nombre>/` **no se borra**: queda como historia de la
exploración, con una nota al inicio que apunta al proyecto y a su registro de
graduación.

**El scaffolding de código no es parte de la graduación.** Llega con la Fase 0
del primer plan de trabajo del proyecto ya graduado.

**Antes del primer plan, el proyecto lee las convenciones y registra cómo las
cumple.** El registro de graduación lleva una sección **Conformidad con las
convenciones**: una tabla con cada regla de [`conventions.md`](conventions.md)
y de `AGENTS.md` que el proyecto va a tocar —idioma, nomenclatura de archivos, de
datos y rutas, de entornos, ramas—, su fuente, y cómo nace el proyecto respecto
de ella. El primer plan de trabajo no se aprueba sin esa sección, y su bloque
«Convenciones aplicables» de la plantilla (precursor, `vc/docs/templates/work-plan.md` en `51fbd2a1`) la cita.

**El código que llega de un prototipo se adapta en la Fase 0; no se adopta.**
Si la exploración dejó código, esquemas o datos —en `ideas/`, en otra carpeta o
en otra herramienta—, la Fase 0 los reescribe para que cumplan las convenciones
antes de construir encima. Revisar su contenido no alcanza: hay que revisar sus
nombres. Un nombre heredado que el PO decide conservar se declara como deuda
con la decisión que lo corrige, según
[«Una convención incumplida no se extiende»](conventions.md#una-convención-incumplida-no-se-extiende).

Precedente que motiva la regla: Melquisedec nació de un prototipo con el
esquema en castellano. Su D0 lo adoptó revisando el contenido pieza por pieza,
pero no los nombres, y tres hitos seguidos los extendieron. La revisión de
conformidad llegó recién con su acta 0003, cuando corregir ya exigía migrar
datos. Hecha en la Fase 0, habría costado un commit.

**Regla explícita para greenfield: el primer código nace dentro de
`vc/<nombre>/`, nunca antes y nunca afuera.** El antipatrón está documentado y
es propio: Pinot Index graduó con gobernanza completa y cero líneas de código, y
su código terminó escribiéndose fuera del proyecto antes de que existiera dónde
ponerlo — corregido después por
su decisión 0002 (precursor, `vc/pinot-index/docs/decisions/0002-repo-extraction.md` en `51fbd2a1`),
que trajo la librería al monorepo. Un proyecto greenfield que gradúa hoy no
repite esa secuencia.

## Mecánica transversal 1 — Vencimiento por estado

Cada estado declara cuánto puede durar sin actividad antes de considerarse
estancado. Estancado no es un veredicto: es una señal de que la idea necesita
volver al gate, aunque sea para archivarla.

| Estado | Vencimiento propuesto | Motivo |
|---|---|---|
| `registered` | Sin límite | Una idea capturada que nadie mira es inventario, no deuda. |
| `exploring` | El propio presupuesto declarado | El timebox ya es el vencimiento; no hace falta un segundo número. |
| `gate-ready` | 60 días | Una definición lista para decidir se desactualiza: el mercado, la capacidad del portfolio y las alternativas cambian. |
| `deferred` | Sin límite, con revisión del disparador | Lo que vence no es la idea sino el disparador: si dejó de poder ocurrir, la idea está archivada de hecho. |
| `archived` | No aplica | — |

**Estos números son defaults ajustables por el PO, no valores sagrados.** El
único que no es arbitrario es el de `exploring`, que se deriva del presupuesto
que el propio PO declaró.

La detección automática **no se implementa hoy**. Queda como ítem de backlog en
`framework-debt.md` (precursor, `vc/docs/standards/framework-debt.md` en `51fbd2a1`) (Ítem 3), en la misma familia que los
deadlines externos que `check-lifecycle` ya vigila. Hasta que exista, el
vencimiento se revisa a ojo — y este documento lo dice en vez de afirmar una
garantía que nadie da.

## Mecánica transversal 2 — Archivar preserva

**La carpeta de una idea archivada no se borra.** Conserva sus documentos, su
evidencia y sus preguntas, y suma una **nota de archivo** al inicio con tres
cosas:

1. **Fecha** del archivo.
2. **Razón**: qué la mató, con la evidencia que lo sostiene.
3. **Qué la reviviría**: la condición concreta bajo la cual volvería a tener
   sentido. Puede ser "nada previsible", dicho explícitamente.

Es el mismo principio de historial inmutable que el framework ya aplica a las
evaluaciones de tooling y a los registros de decisión, aplicado a ideas: el
valor de una idea muerta es que la próxima idea parecida no repita la
investigación que la mató.

## Mecánica transversal 3 — Estados visibles

**El estado vive en el documento de idea**, en su sección "Estado" — que es
donde la convención de `ideas/` ya pone estado, prioridad, versión y sprint. No
se introduce frontmatter: la ficha no lo usa hoy y agregarlo cambiaría la forma
de todos los documentos existentes para ganar nada.

**En el archivo de seguimiento**, el estado del ciclo viaja como tag
`lifecycle:<estado interno>` — por ejemplo `lifecycle:registered`. `tags` es una
lista de strings libre en el schema vigente, así que esto **no requiere ningún
cambio de schema**: ni versión nueva del archivo de seguimiento, ni migración de
los existentes.

El tablero muestra ese tag como etiqueta en la tarjeta de la idea. Los seis
estados del ciclo son ortogonales al `status` del tablero (`exploring`,
`planned`, ...), que sigue significando lo que significaba: una idea
`gate-ready` puede seguir siendo `exploring` para el tablero sin contradicción.

Lo que **no** se hizo, por la misma razón que el vencimiento automático: nada
valida que el tag exista, que sea uno de los seis, ni que coincida con lo que
dice la ficha. Mismo ítem de backlog.

## Aplicación

| Idea | Estado | Nota |
|---|---|---|
| `terroir` (precursor, `ideas/terroir/README.md` en `51fbd2a1`) | `exploring` desde 2026-08-23; **en hold hasta 2026-10-05** | Primer usuario del ciclo. Cuatro asesinas declaradas. Presupuesto de exploración (2 semanas, 2026-08-23 a 2026-09-06) declarado por el PO y **vencido sin cerrar**: el 2026-09-05 el PO decidió un hold de 30 días en vez de las tres salidas del paso 2. No se emiten sesiones hasta el vencimiento; ahí se retoma la validación de dos lados, se extiende o se archiva. |
| `salmanazar` (precursor, `ideas/salmanazar/README.md` en `51fbd2a1`) | `graduated` (2026-10-24) | Registrada el 2026-08-29. Presupuesto de exploración declarado por el PO el 2026-09-05: **2 semanas, ventana 2026-09-08 a 2026-09-22**, con lo que entró al paso 2; en hold desde el 2026-09-25. Redefinida por el PO el 2026-10-24 como dueña de la ingesta de etiquetas, del orden de la cava y de la ventana de consumo, con Teyst como proveedor de identidad y Melquisedec leyendo su base central. **Graduada el 2026-10-24** (acta (precursor, `vc/salmanazar/docs/decisions/0001-graduation.md` en `51fbd2a1`)): nace `vc/salmanazar/`, con los proyectos `dev-<otro>` y `prod-<otro>`. Las asesinas 1, 2 y 4 —para quién es, si se sostiene el hábito de carga y qué la hace mejor que Vivino, CellarTracker o una planilla— llegaron al gate sin evidencia: el PO lo asentó como desviación deliberada, con el precedente de Brix. La 3 —de dónde salen los datos del vino— tiene la evidencia de la prueba del PO del 2026-10-24 sobre `/upload` de Melquisedec, en iPhone y Mac. |
| `brix` (precursor, `ideas/brix/README.md` en `51fbd2a1`) | `graduated` (2026-09-13) | Registrada el 2026-09-02. Presupuesto de exploración declarado por el PO el 2026-09-05: **2 semanas, ventana 2026-09-05 a 2026-09-19**, con fecha de inicio la del merge de #207 (PR 207 del precursor). Primera idea del pipeline sobre el propio framework y no sobre un producto: si gradúa, lo que nace son cambios de estándar y tooling, no un `vc/<nombre>/`. Produce tres entregables atados a sus tres asesinas: veredicto sobre la medición de consumo por entregable, la unidad de resultado del portfolio, y el punto de equilibrio del ahorro de tokens. **Graduada el 2026-09-13** (acta (precursor, `vc/brix/docs/decisions/0001-graduation.md` en `51fbd2a1`)) bajo la lectura de este estándar, que hace nacer `vc/brix/`, y no la de la ficha. Las asesinas 1 y 3 y el diagnóstico de C5 llegaron al gate sin evidencia completa: el PO lo asentó como desviación deliberada. |
| `clarificacion` (precursor, `ideas/clarificacion/README.md` en `51fbd2a1`) | `registered` desde 2026-09-06 | Segunda idea sobre el propio framework. Nace de comparar Blends con la arquitectura de agentes de Spotify (hook PreToolUse que bloquea lecturas y las redirige a un worker barato) y de una medición propia sobre 90 sesiones: el 42 % del contexto entra por `cat`/`sed` en Bash y solo el 8 % por resultados de más de 350 líneas. Tres asesinas declaradas; presupuesto de exploración pendiente, junto con la decisión del PO de si vive sola o se pliega a Brix, que explora el mismo punto desde la unidad de resultado. |
| `melquisedec` (precursor, `ideas/melquisedec/README.md` en `51fbd2a1`) | `graduated` (2026-08-30) | Nacida como **Vinograma** el 2026-08-28 y renombrada el 2026-08-29 (decisión del PO). Regularización retroactiva: la idea nació con scaffolding de código ya construido fuera del pipeline (sesión de Cowork), regularizada en #122 (PR 122 del precursor)/#124 (PR 124 del precursor)/#126 (PR 126 del precursor) — documentos movidos a la carpeta de la idea; código y datos quedaron congelados fuera de blends, citados solo como evidencia. Cuatro asesinas y presupuesto de exploración (2 semanas) confirmados por el PO el 2026-08-28. **Graduada el 2026-08-30** (acta (precursor, `vc/melquisedec/docs/decisions/0001-graduation.md` en `51fbd2a1`)); D0 en `main` desde #191 (PR 191 del precursor). |
| `teyst` (precursor, `ideas/teyst/README.md` en `51fbd2a1`) | `graduated` (2026-08-13) | Precedente brownfield. Su paso 3 fue el assessment de la Fase 1. |
| `pinot-index` (precursor, `ideas/pinot-index/README.md` en `51fbd2a1`) | `graduated` (2026-08-13) | Precedente greenfield, y origen de la regla del paso 5 sobre dónde nace el primer código. |
| `trasiego` (precursor, `ideas/trasiego/README.md` en `51fbd2a1`) | `exploring` desde 2026-09-15; **`gate-ready` desde 2026-10-02** | Integración MCP con Vivino para traer datos reales de vinos (bodega, varietal, región, añada, puntaje comunitario) en vez de carga 100% manual. Nace pensando en alimentar el catálogo de vinos del ecosistema de catas de Teyst — sin asumir todavía si Teyst la consume directo, si pasa por un catálogo intermedio, o si vive como servicio propio (misma disciplina que `salmanazar` con la dependencia de Melquisedec: pregunta abierta, no acoplamiento asumido). Tras la segunda validación (X-Wines, 2026-10-02) el PO decidió una planificación acotada: PRD-lite, MVP = script local de semilla de curaduría para Melquisedec. La recomendación al gate (T-04, SP-2026-W42, decide el PO) es absorber el MVP en Melquisedec y archivar la idea; graduar `vc/trasiego/` solo con un segundo consumidor. Pendiente antes del gate: la procedencia de las valoraciones (T-03). |

Teyst y Pinot Index se listan por su valor de precedente. No se les aplica
retroactivamente el ciclo: graduaron antes de que existiera, y reescribir su
historia para que parezca que lo siguieron sería exactamente el tipo de
afirmación falsa que el framework registra como deuda. Melquisedec no necesita
esa aclaración: es la primera idea que recorrió el ciclo entero, del paso 1 al
5, y su fila describe lo que efectivamente pasó.

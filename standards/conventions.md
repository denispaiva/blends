# Convenciones de codificación y nomenclatura

Cómo se nombra, se escribe y se commitea en cualquier proyecto del portfolio. Extiende la regla de idioma de `AGENTS.md` (precursor, `AGENTS.md` en `51fbd2a1`) con la nomenclatura concreta que esa regla no fija.

## Principio

La frontera no es técnica sino de audiencia: **inglés para lo que interpreta una máquina, castellano para lo que razona una persona**. Un identificador es un símbolo; un comentario es una explicación. Por eso `eventStateMachine.ts` convive con un comentario que dice *"Auth es la fuente de verdad"*, y ambos están bien.

El mensaje de commit cae del lado humano: dentro de un año nadie lee la historia buscando sintaxis, la lee buscando por qué se hizo algo.

## Idioma por artefacto

| Artefacto | Idioma | Motivo |
| --- | --- | --- |
| Código, identificadores, claves de esquema, estados internos | Inglés | Los interpreta una máquina o los consume otro sistema |
| Nombres de archivo y carpeta | Inglés | Son rutas, no prosa |
| Colecciones, campos, valores de enum y rutas de URL | Inglés | Los persiste una base de datos o los enruta un navegador; forma en [Nomenclatura de datos y rutas](#nomenclatura-de-datos-y-rutas) |
| Ramas | Inglés | Las lee el sistema de integración |
| Comentarios en el código | Castellano | Explican el porqué a una persona |
| Documentación, ADR, specs, planes | Castellano | Escritos para leerse |
| **Mensajes de commit** | **Castellano**, salvo el prefijo `tipo(alcance):` | Narrativa histórica con un prefijo estructurado que sí interpretan herramientas |
| Mensajes de error e interfaz | Castellano | Los lee quien usa el sistema |
| Invocación de un comando slash (`.claude/commands/*.md`) | Inglés | Es nombre de archivo, id del registro de capacidades y lo que escribe quien invoca, las tres cosas a la vez |

Los nombres propios de productos, bibliotecas, protocolos y comandos conservan su forma oficial en cualquier contexto.

**Un comando slash es una excepción declarada, no una regla nueva.** Su invocación (`/scarcity`, no `/escasez`) cae del lado inglés aunque la interpreta una persona, porque es simultáneamente tres cosas que sí interpreta una máquina: el nombre del archivo en `.claude/commands/`, el id `slash.<nombre>` que exige el registro de capacidades (`vc/tooling/brix-capabilities.json`, regla R8 de `validate-brix-capabilities.mjs`), y lo que un alias normaliza para enrutar. El castellano sigue siendo la explicación: descripción, contenido del comando y summary del registro. Precedente: `/escasez` y `/escasez-cierre` se renombraron a `/scarcity` y `/scarcity-closure` el 2026-09-27; los archivos viejos quedan como stub `deprecated` hasta el próximo tren, no se borran en el mismo commit que los reemplaza.

## Nomenclatura de archivos

| Ámbito | Patrón | Ejemplo |
| --- | --- | --- |
| Documentos | `kebab-case.md` | `product-invariants.md` |
| Documentos institucionales | `MAYÚSCULAS.md` | `README.md`, `AGENTS.md`, `CHANGELOG.md`, `LICENSE` |
| Decisiones de arquitectura | `ADR-NNN-kebab-case.md` | `ADR-002-modelo-de-roles.md` |
| Specs y planes | `AAAA-MM-DD-tema.md` | `2026-08-06-ci-cd-strategy.md` |
| Módulos de código | `camelCase` + extensión | `eventStateMachine.ts` |
| Componentes y clases | `PascalCase` + extensión | `ActiveSessionCard.tsx` |
| Pruebas | `<área>-<caso>.test.<ext>` | `e3-3-reveal-capture.test.ts` |
| Configuración | El nombre que exige la herramienta; si es libre, `kebab-case` | `eslint.config.js`, `.nvmrc` |
| Identificadores en JSON | `kebab-case` | `google-one-2tb` |

Tres precisiones que evitan la deriva observada en proyectos existentes:

- **Un solo estilo por carpeta.** Que `docs/` tenga `PROJECT_BRIEF.md` junto a `product-invariants.md` obliga a adivinar el nombre al buscar.
- **El número de ADR nunca se reutiliza**, ni siquiera si la decisión se descarta: se marca como reemplazada y se escribe una nueva.
- **El nombre de una prueba declara a qué responde.** Un archivo llamado `f09-resolution-idempotency.test.ts` dice de qué épica salió; `test2.ts` no dice nada.

## Nomenclatura de datos y rutas

La regla de idioma ya cubría las claves de esquema y los estados internos, pero no decía con qué forma se escriben ni nombraba las bases de datos ni las URL. Esta tabla lo fija para todo lo que se persiste, se sirve o se enruta:

| Ámbito | Idioma | Patrón | Ejemplo |
| --- | --- | --- | --- |
| Colecciones de Firestore (o tablas de otro backend) | Inglés | `snake_case`, plural | `events`, `catalog_labels` |
| Campos de un documento persistido | Inglés | `snake_case` | `event_id`, `created_at`, `country_id` |
| Valores de enum y estados persistidos | Inglés | `snake_case` | `in_progress`, `curated`, `red` |
| IDs de documento que son slug | Inglés si son técnicos; el nombre propio conserva su forma | `kebab-case` ASCII | `ig-lujan-de-cuyo`, `malbec` |
| Rutas de URL y archivos servidos | Inglés | `kebab-case` | `/catalog.json`, `/labels/` |
| Variables y propiedades en el código | Inglés | `camelCase` | `eventId` |

- **El precedente es Teyst.** Sus colecciones y campos ya son inglés en `snake_case` (`events`, `event_id`, `community_memberships`), y Teyst consume el catálogo de Melquisedec: un contrato entre dos proyectos no puede tener dos formas.
- **El valor interno va en inglés y la interfaz lo traduce.** Es la misma regla que `in_progress` → «En curso», aplicada a cualquier enum: `curated` se guarda, «Curada» se muestra. Una traducción guardada en la base de datos no es un valor interno, es copy en el lugar equivocado.
- **El código puede adaptar la forma, no el idioma.** Un tipo de TypeScript que mapea un documento puede exponer `eventId` sobre un campo `event_id`; lo que no puede es llamarlo `eventoId`.
- **Una ruta de URL es una clave, no copy.** La leen el navegador, el service worker, la CSP y la caché; el texto que ve la persona es el título de la página.
- **Deuda conocida, no licencia.** En las Functions de Teyst conviven `eventId` y `event_id` como claves de objeto (15 contra 53 al 2026-10-07; falta verificar cuáles se persisten); Melquisedec nació con colecciones, campos, enums, carpetas y rutas en castellano (`catalogo_etiquetas`, `paisId`, `curada`, `/catalogo.json`). Ninguno de los dos casos sirve de precedente: se corrigen, y Melquisedec lo hizo junto con su migración al proyecto `dev-<token>`. Sus nombres estuvieron declarados uno por uno en `vc/tooling/data-naming.json` (precursor, `vc/tooling/data-naming.json` en `51fbd2a1`); la entrada se borró el 2026-10-10, cuando el contrato v2 (acta 0003 de Melquisedec) terminó el renombre y el validador pasó sin tolerancia.

**Caso fundante.** El esquema de Melquisedec nació el 2026-08-28 en una sesión fuera del circuito de ideas, copiando los nombres del prototipo. El D0 (2026-08-30) lo adoptó revisando el contenido pieza por pieza, pero no la forma de los nombres, y cada hito posterior extendió el patrón que encontró: la coherencia con el código existente le ganó a una regla que no lo nombraba y que nada verificaba.

## Una convención incumplida no se extiende

**El código existente que viola una convención no es precedente: es deuda.** Vale para cualquier regla de esta página, no solo para la nomenclatura de datos. Quien la encuentra hace tres cosas:

1. **No la extiende.** Lo nuevo nace cumpliendo la regla, aunque conviva con lo viejo que no la cumple. «Se respeta el vigente» no es motivo para crear un nombre más que incumple.
2. **La declara.** En el registro de deuda que corresponda (`data-naming.json`, `environment-ids.json`) o, si la regla no tiene registro, en el plan que la encontró, con la decisión que la va a corregir.
3. **No la corrige por su cuenta.** Renombrar datos desplegados es una migración y la decide el PO; lo que sí corresponde es proponerla.

**Por qué es regla y no consejo.** En Melquisedec la regla de idioma estaba escrita en `AGENTS.md`, que se carga en cada sesión que toca `vc/`, y aun así tres hitos seguidos extendieron nombres en castellano: cada sesión priorizó la coherencia con el código que encontró. Leer la regla no alcanzó; hace falta que la regla diga qué hacer cuando el código la contradice. Teyst ya lo hacía con el renombre de `community_*` en su `AGENTS.md`; esta sección lo generaliza.

## Ramas y commits

Ramas: prefijo, barra, y `kebab-case` descriptivo — `feat/ci-cd-strategy`.

El trabajo de **etapa de idea** usa su propio prefijo: `idea/<slug>`, con el slug de la carpeta en `ideas/` — `idea/terroir`. Una idea todavía no es una feature: nombrar `feat/` a la captura de una idea afirma un compromiso de construcción que el pipeline no tomó todavía. El ciclo completo está en [`idea-lifecycle.md`](idea-lifecycle.md).

### Prefijos en uso

Hasta el 2026-08-23 este documento nombraba cuatro prefijos (`feat/`, `fix/`, `chore/`, `experiment/`) mientras el historial usaba nueve. La lista de abajo es el inventario real, contado sobre los PRs del repositorio, no la lista que hubiéramos querido tener.

| Prefijo | Para qué | Uso real |
| --- | --- | --- |
| `idea/<slug>` | Etapa de idea | La captura y exploración de una idea de `ideas/` |
| `feat/`, `fix/` | Trabajo de proyecto o de framework | El grueso del historial |
| `docs/` | Cambios de documentación | Registro de decisiones, correcciones de docs |
| `tooling/` | Scoring Skill | Evaluación, adopción y promoción de herramientas |
| `finops/` | Ledger FinOps | Sincronización de ítems y actuals |
| `chore/` | Framework | Los cambios recientes del framework mismo |
| `hotfix/` | Urgencia sobre `main` | Excepcional |
| `experiment/` | Exploración descartable | Declarado, sin uso todavía |

Cuando el trabajo pertenece a un proyecto, el slug empieza con el token del proyecto: `feat/teyst-fase-3-1-fundacion`, `docs/teyst-fase-4-5-porteo-cliente`, `feat/pinot-extract-scoring-engine`. Ese token es lo que permite derivar el alcance sin consultar a nadie.

### Alcance de escritura por rama

El nombre de la rama declara qué puede tocar. El mapa vive en `vc/tooling/branch-scopes.json` (precursor, `vc/tooling/branch-scopes.json` en `51fbd2a1`) y lo verifica `vc/tools/check-branch-scope.mjs` en cada PR:

| Rama | Alcance |
| --- | --- |
| `idea/<slug>` | `ideas/<slug>/**`, con el slug resuelto contra las carpetas que existen en `ideas/` — `idea/terroir-exploracion` alcanza `ideas/terroir/**` |
| `<prefijo>/<proyecto>-*` | `vc/<proyecto>/**` y `ideas/<proyecto>/**` — los documentos del assessment brownfield viven en `ideas/` |
| `tooling/*` | `vc/tooling/**`, `vc/docs/tooling/**`, `vc/tools/**` y el archivo de seguimiento del proyecto que adopta |
| `finops/*` | `vc/finops/**` y el schema y validador del ledger |
| Resto de los prefijos, sin token de proyecto | Framework: `vc/docs/**`, `vc/tools/**`, `vc/tooling/**`, `vc/framework/**`, `vc/finops/**`, `roadmap/**`, `docs/**` y los archivos de raíz (incluido `.github/`) |
| `tooling/*` y ramas de framework | Config de sesión de Claude Code: `.claude/settings.json` en ambas, más `.claude/commands/**` en `tooling/*` y `.claude/hooks/**` en las de framework. El mapa ya lo admitía (`52a18cc`, vagón 1 de este ciclo): esta fila documenta esa regla, no la introduce. El hook `Stop` que no deja cerrar una sesión sin entregar vive en `vc/tools/hooks/stop-deliver.mjs` —tooling del framework, no config—; `.claude/hooks/**` queda como ruta admitida sin uso |

Los generados del dashboard (`roadmap/data.js`, `roadmap.html`) están permitidos en toda rama: cualquier cambio de una fuente de seguimiento los regenera.

**Un conflicto en un archivo generado se resuelve regenerando, nunca a mano.** Es la contracara del permiso de arriba: si toda rama puede regenerarlos, dos ramas que lo hagan van a chocar en las mismas líneas de un archivo que ninguna de las dos edita. La resolución no depende de qué cambió cada lado —mergear la fuente, correr el generador, commitear el resultado— y por eso no hace falta leer el conflicto para resolverlo:

```bash
git merge origin/main          # la fuente (`.roadmap.json`) mergea sola
node roadmap/build.mjs         # el generado se rehace desde la fuente ya mergeada
git add roadmap/data.js && git commit
```

Resolver el conflicto a mano produce un archivo que **ningún generador emitiría**: se ve bien en el diff y falla en el próximo `--check`. El caso que fija la regla es la PR #229 (PR 229 del precursor) (2026-09-05), que regeneró `data.js` con el generador viejo mientras la #226 (PR 226 del precursor) cambiaba ese mismo generador. La regeneración devolvió el diff a los 2 archivos esperados sin leer una sola línea del conflicto. El arreglo de fondo, que el conflicto no llegue a ocurrir, es el Ítem 12 de `framework-debt.md` (precursor, `vc/docs/standards/framework-debt.md` en `51fbd2a1`); esta regla vale mientras ese ítem siga abierto.

**El conflicto no avisa.** Una PR verde pasa a `DIRTY` en silencio cuando `main` avanza, y GitHub no notifica. Entre abrir una PR y mergearla, `gh pr view <n> --json mergeStateStatus` es la única forma de enterarse sin mirar la página.

Una rama de idea puede llevar sufijo para nombrar el trabajo concreto —
`idea/terroir-exploracion`, `idea/terroir-gate` — y sigue alcanzando la carpeta
de la idea. El slug se resuelve contra las carpetas que existen en `ideas/` **en
el árbol base contra el que se compara el diff**, tomando el prefijo más largo
que coincida en frontera de guión. Salen del árbol base y no del working tree a
propósito: si salieran del working tree, una rama podría ensanchar su propio
alcance creando una carpeta. Un slug sin carpeta —una idea nueva, cuya carpeta
todavía no está en el árbol base— conserva el alcance literal y produce
advertencia.

**Por qué existe la regla.** La [regla 2 de paralelismo](role-matrix.md) exige `writeScope` disjunto entre tareas, pero hasta ahora eso vivía solo en el plan de una sesión. El nombre de la rama es el único dato que dos sesiones que no se conocen comparten de antemano: derivar el alcance de ahí es lo que hace verificable el `writeScope` disjunto sin coordinación previa.

**Un proyecto que toca la CI compartida usa excepción declarada, siempre** (decisión del PO, 2026-08-23). Los proyectos **no** tienen `.github/**` en su alcance, y no se lo va a agregar: agregar un job propio o ajustar un pipeline es legítimo pero **ocasional**, y abrirlo en el mapa sería un agujero permanente por un caso esporádico. La excepción deja cada cruce visible y auditable — que es precisamente lo que se quiere cuando un proyecto modifica infraestructura que comparte con todos los demás.

**Excepción declarada.** Una rama puede exceder su alcance si el desvío es deliberado. Se declara en un archivo `SCOPE-EXCEPTION.md` en la raíz, **commiteado dentro del mismo diff**, con dos campos obligatorios:

```markdown
Razón: por qué este trabajo necesita salirse de su alcance.
Decisión: ruta o enlace al registro de decisión que lo autoriza.
```

Con la excepción presente y completa, el check pasa con **advertencia visible** que enumera los archivos y cita la razón; nunca en silencio. Sin ella, falla. Con la excepción incompleta —le falta alguno de los dos campos— falla también: un desvío sin razón ni decisión que lo autorice es exactamente lo que la regla existe para evitar.

Se eligió un archivo commiteado y no un marcador en la descripción del PR porque el archivo es lo verificable: viaja en el diff que se revisa, queda en la historia, y no puede editarse después de que el check pasó. Una descripción de PR es texto mutable fuera de la historia de git y exige una llamada a la API para leerlo.

Una rama cuyo prefijo no está en la tabla produce **advertencia, no error**: hay ramas históricas anteriores a esta convención, y endurecerlo es decisión del PO.

Commits: Conventional Commits con el cuerpo en castellano.

```
tipo(alcance): qué cambió, en imperativo y en castellano

Por qué cambió y qué decisión sostiene, si no es evidente en el título.
Las consecuencias que quien lea la historia necesita conocer.

Co-Authored-By: <agente> <correo>
```

- **Tipo**, en inglés: `feat`, `fix`, `chore`, `docs`, `test`, `refactor`, `perf`, `build`, `ci`.
- **Alcance**, en inglés: el área o la épica — `finops`, `ci`, `e3.3`, `t1`.
- **Título**: imperativo, en castellano, sin punto final, hasta unos 72 caracteres.
- **Cuerpo**: obligatorio cuando el cambio no se explica solo, y siempre que la decisión sea discutible.
- Un commit describe **un resultado concreto** y no mezcla proyectos sin necesidad.
- El mensaje describe lo que el commit **hace de verdad**: anunciar un cambio que no ocurrió es una mentira en la historia, y la historia no se corrige después.

### La entrega nombra su tarea: el `qualifiedId` completo

**Todo commit o PR que entrega trabajo de una tarea del roadmap nombra el `qualifiedId` completo de esa tarea** —`<proyecto>/T-nn`, el `id` del `.roadmap.json` del proyecto seguido del id local de la tarea, tal como lo califica `roadmap/build.mjs`— en el cuerpo del mensaje del commit o en el cuerpo del PR. Es la decisión D-8 del PO (2026-10-28), compuerta del spike de la Fase 0 del Backlog como stack: `docs/superpowers/specs/2026-10-28-backlog-visual-design.md`, §7.

- **Un `T-nn` a secas no es un vínculo.** Diez roadmaps numeran sus tareas `T-nn` —los seis proyectos de `vc/` y cuatro ideas de `ideas/`— y el mismo número existe en varios: `T-20` no identifica nada; `blends-framework/T-20` identifica una sola tarea del portfolio. El prefijo válido es el `id` del `.roadmap.json` que `build.mjs` recoge, el mismo que muestra `roadmap/data.js`: al 2026-10-28, `teyst`, `melquisedec`, `brix`, `pinot-index`, `salmanazar` y `blends-framework` entre los proyectos, y `clarificacion`, `decantar`, `terroir` y `trasiego` entre las ideas.
- **Una entrega puede nombrar varias tareas**, cada una con su `qualifiedId` completo. Un commit que no entrega trabajo de ninguna tarea —un fix de validador, un bump de versión, una corrección de docs— no nombra ninguna, y eso es correcto: la regla no obliga a inventar una tarea, obliga a no entregar una sin nombrarla.
- **En el cuerpo, no en el título.** El título conserva la forma `tipo(alcance): asunto`; el `qualifiedId` va en el cuerpo del commit o, cuando la entrega entra por PR individual (órdenes 1 y 2 de `github-scarcity.md` (precursor, `vc/docs/standards/github-scarcity.md` en `51fbd2a1`)), en el cuerpo del PR. Un vagón del tren se mergea con su historia completa, así que el cuerpo de sus commits alcanza: el cuerpo del PR del tren lo escribe `train.mjs`, no quien entrega.
- **Rige desde la versión 1.103.0 de este estándar y no es retroactiva.** Las 58 tareas que ya estaban `done` el 2026-10-28 no se reconstruyen: quedan como cierre, nunca como entrega verificada; las 33 que nacieron terminadas llevan además la marca «registrada terminada» de la vista. Agregar el vínculo reescribiendo commits ya mergeados está prohibido por la regla de arriba: la historia no se corrige después.

**Quién la lee.** El proyector del Backlog, en su Fase 3b, cuando exista: busca con `git log --grep '<qualifiedId>'` sobre la historia que cada PR mergeó en `origin/main` y asciende la tarjeta de `closure_in_main` a `delivery_in_main` solo si el merge con checks contiene al menos un commit, o el cuerpo del PR, con ese `qualifiedId` (invariante I-3 de la spec). Hasta entonces nadie la lee de forma automática. La convención llega antes que su lector a propósito: el proyector solo puede mostrar lo que la historia acumuló, y sin la regla no acumula nada.

**Qué no la verifica, y por qué.** El hook `commit-msg` no la exige ni avisa. No toda rama entrega una tarea, así que el hook no puede saber cuándo falta el vínculo, y una advertencia en cada commit sin tarea sería ruido que entrena a ignorarla. Queda para la revisión, como la honestidad del mensaje de commit. Si el proyector muestra que la convención no se cumple, pasar a verificarla es decisión del PO.

**Motivo.** El spike de la Fase 0 (`docs/superpowers/specs/2026-10-28-backlog-visual-spike/reporte-cierres-2026-10-28.md`, medido sobre `main` al 2026-10-28) encontró que el cierre administrativo se resuelve —58 de 58 tareas terminadas tienen commit de cierre, 51 de 58 con PR— pero ninguna fuente vincula una tarea con el entregable de su trabajo: 85 de 2.200 commits nombran una `T-nn`, 67 de ellos con el proyecto, y solo 7 tocan código; el resto son commits de roadmap y documentación. Sin esta convención, el verde pleno de la vista no tiene datos de dónde salir. La alternativa descartada era declarar las tareas en la entrada de la cola de merge (`tasks: ["teyst/T-27"]`), que el tren conserva en `archive/`; queda descrita en ese mismo reporte. El PO eligió el mensaje del commit: lo escribe quien hizo el trabajo, viaja con el merge y se busca con `git log`, sin leer la rama `queue`.

**Ejemplo.** Una rama que entrega la tarea `T-27` de Teyst:

```text
feat(teyst): el hosting de prod sirve la app en teyst.web.app

Entrega teyst/T-27: despliegue de hosting a prod-<proyecto> tras el gate de 3 en
canary. El backend ya estaba desplegado desde el tren #270.

Co-Authored-By: <agente> <correo>
```

`git log --grep 'teyst/T-27'` lo encuentra; `git log --grep 'T-27'` encontraría también cualquier `T-27` de los otros nueve roadmaps.

## Nomenclatura de entornos

Dos reglas, una para los nombres que ya existen y otra para los que nacen.

**Regla: nunca se nombra un entorno solo por su ID técnico.** Toda instrucción, documento, prompt o reporte que mencione un entorno usa **el apodo inequívoco junto al ID** — "el legacy (`<proyecto>-dev`)", "el nuevo dev (`dev-<proyecto>`)", "el nuevo prod (`prod-<proyecto>`)".

Aplica a las cuatro superficies por igual: lo que un agente lee, lo que un agente escribe, lo que el PO pega en un prompt y lo que un reporte devuelve. Un ID solo, sin apodo, es un incumplimiento aunque el contexto parezca obvio — el contexto no viaja con la línea cuando alguien la cita en otro lado.

### Ejemplo: los tres entornos de un proyecto en migración

| Apodo obligatorio | ID técnico | Qué es |
| --- | --- | --- |
| **el legacy** | `<proyecto>-dev` | El sistema **viejo**, con usuarios reales hoy y el canario desplegado. **Solo lectura salvo autorización explícita.** |
| **el nuevo dev** | `dev-<proyecto>` | Entorno de desarrollo del stack nuevo. |
| **el nuevo prod** | `prod-<proyecto>` | Producción futura del stack nuevo; cutover pendiente. |

**Por qué esta tabla vive en el estándar del framework y no en el proyecto:** existe justamente para quien **no** conoce el proyecto. Un agente que llega sin contexto lee las convenciones antes que la documentación de Teyst, y es exactamente ese lector el que puede operar contra el entorno equivocado.

### El motivo, sin suavizarlo

`<proyecto>-dev` y `dev-<proyecto>` son la misma palabra con las partes al revés. No se distinguen de un vistazo, no se distinguen al dictarlas, y no se distinguen en un `grep` apurado. Y lo que hay del otro lado no es simétrico: uno tiene usuarios reales y el otro es descartable.

**Caso fundante:** el propio PO los cruzó en conversación el **2026-08-24**. Si el que creó los proyectos los confunde, un agente sin contexto también — y el error más caro que se puede cometer acá no es romper el build, es operar contra el entorno equivocado.

La regla no arregla los nombres, que ya están puestos y renombrar proyectos de Firebase no es gratis. Que no se repita para lo que nace lo fija la regla de prefijo, más abajo. Lo que hace es **obligar a que el nombre nunca viaje solo**: el apodo carga el significado que el ID no distingue.

### Cómo nace un ID de entorno: el entorno va primero

**Regla obligatoria desde el 2026-10-06:** todo proyecto que nace en Firebase, en Google Cloud o en cualquier otro backend se nombra **`<entorno>-<token>`**. El entorno va como prefijo y el token del proyecto después: `dev-<proyecto>`, `prod-<proyecto>`, `dev-<token>`. Nunca `<proyecto>-dev` ni `<token>-dev`.

| Parte | Valores | Regla |
| --- | --- | --- |
| Entorno | `dev`, `staging`, `prod` | Siempre primero. Sumar otro entorno es un cambio de este estándar, no una decisión del proyecto. |
| Token | El del proyecto, en `kebab-case` | Es el mismo en todos los entornos del proyecto: `dev-<token>` y `prod-<token>`, no `dev-<token>` y `prod-<nombre-largo>`. |
| Desambiguador | Opcional, al final | Solo si el ID está tomado, porque los IDs de GCP son globales: `dev-<token>-<desambiguador>`. El entorno no se mueve del principio. |

La regla cubre el ID del proyecto, su nombre visible y todo sitio de Hosting o recurso que lleve el entorno en el nombre. El sitio por defecto de Hosting hereda el ID del proyecto, así que cumple solo. Un sitio de producción con el nombre pelado del producto, como `teyst`, es válido: no lleva entorno, así que no hay orden que respetar. Los proyectos `demo-*` del Emulator Suite ya cumplen, porque Firebase exige ese prefijo.

**Por qué el prefijo:** con el entorno adelante, la lista de proyectos se agrupa por entorno, y lo primero que se lee es si lo que se va a tocar es producción. Y cierra la confusión de la sección anterior: si todo lo que nace es `dev-…`, cualquier `…-dev` es heredado por construcción, y el par `<proyecto>-dev` contra `dev-<proyecto>` no se puede repetir.

**Un ID de proyecto de GCP no se renombra.** Corregir un heredado no es editar un nombre: es crear el proyecto con el ID correcto y migrar base, Hosting, identidad de despliegue, presupuesto y cuentas de servicio. Esa migración la decide el PO caso por caso. Mientras tanto el heredado se sigue nombrando con su apodo, como pide la regla de arriba.

#### Ningún backend nace sin su fila

**Regla obligatoria desde el 2026-10-26:** ningún proyecto de backend se crea en Google Cloud ni en Firebase sin su fila en la lista `active` de `environment-ids.json` (precursor, `vc/tooling/environment-ids.json` en `51fbd2a1`). La fila declara el ID, el proyecto de `vc/` dueño, el entorno, el estado, la fecha de alta y quién lo creó.

- **La fila exige un proyecto graduado.** `vc/<dueño>/.roadmap.json` tiene que existir en el mismo árbol que se valida. Un backend nunca pertenece a una idea: una idea que necesita backend gradúa primero. Como el chequeo mira el árbol y no la cola, una fila vale apilada sobre la rama de graduación y vale en `main` después del tren.
- **Dos estados.** `planned` es la fila escrita antes de crear el proyecto, sin fecha ni autor: un backend decidido y todavía no creado es un estado legítimo y visible. Al crearlo, el PO pasa la fila a `active` con la fecha de alta (el `createTime` de `gcloud projects describe`) y quién lo creó.
- **El registro manda.** `environment-ids.json` es la fuente normativa; la tabla de abajo es su vista humana y el chequeo exige que coincidan, fila por fila, en ID, proyecto y estado.
- **Lo que no es backend de Blends también se declara.** Un proyecto de GCP de las cuentas del PO que no es backend de ningún proyecto de `vc/` va en la lista `external`, o en `externalPatterns` si lo crea Google con un sufijo propio.
- **La nube se concilia por mes.** El repo solo ve lo que cita. Lo que existe de verdad en Google Cloud se compara contra el registro con el runbook `backend-inventory.md` (precursor, `vc/docs/tooling/backend-inventory.md` en `51fbd2a1`), que corre el PO porque exige su cuenta.

Motivo: hasta esta regla, un proyecto creado desde la consola era invisible hasta que un `.firebaserc` llegaba a `main`, y nada comparaba los proyectos de la nube con los proyectos graduados. Lo frenaba un procedimiento (los proyectos los crea el PO), no un control. Spec: `2026-10-24-vc-carpetas-orden-design.md` (precursor, `docs/superpowers/specs/2026-10-24-vc-carpetas-orden-design.md` en `51fbd2a1`), Fase S.

#### Inventario

Cada instancia mantiene el inventario de sus entornos (IDs que cumplen, heredados y migraciones) en su propio repositorio, no en el método.

## Sesiones de trabajo

Quién opera una rama, y hasta cuándo existe, está en `sessions.md` (precursor, `vc/docs/standards/sessions.md` en `51fbd2a1`): las dos clases de sesión (permanentes por frente, temporales por tarea), la taxonomía `tipo-acceso-cepa` de las temporales, y sus reglas de vida.

La relación con esta página es directa: el **alcance de escritura por rama** de arriba verifica qué toca cada rama; el estándar de sesiones define **quién** la opera. El ámbito de una sesión no va en su nombre justamente porque ya está declarado en su prompt de apertura y vigilado por `check-branch-scope` — una tercera copia de la misma afirmación sería deuda, no claridad.

## Markdown

- **Un solo H1 por documento**, que es su título.
- Oraciones cortas. Si una oración necesita dos comas para sostenerse, probablemente sean dos oraciones.
- **Tablas para matrices y comparaciones**, nunca para prosa: una tabla con párrafos adentro es una lista disfrazada.
- Bloques de código **siempre con lenguaje declarado**, para que el resaltado funcione.
- **Enlaces relativos** entre documentos del repositorio; absolutos solo hacia afuera.
- **Prohibido publicar un documento con `TBD` o `TODO`.** Si falta algo, se escribe qué falta, por qué y quién lo resolverá.
- Las afirmaciones verificables citan su fuente: un importe lleva su comprobante, una versión lleva dónde se leyó.

## Specs

Una spec de `docs/superpowers/specs` empieza con un bloque de front matter con cinco campos: `id` (el nombre del archivo sin `.md`), `status` (`draft`, `proposed`, `approved`, `implemented`, `superseded` o `abandoned`), `level` (`N1`, `N2` o `N3`, el nivel de [proporcionalidad](work-proportionality.md)), `decides` (quién aprueba, obligatorio desde `approved`) y `plan` (ruta del plan asociado, obligatoria desde `implemented`). Sus requisitos se numeran `- **R-1**`, sin repetir, y se citan desde afuera como `<id-de-la-spec>#R-1`. Las specs con fecha anterior al 2026-10-19 no se obligan a migrar. El motivo y las decisiones están en el diseño (precursor, `docs/superpowers/specs/2026-10-18-specs-contrato-y-validador-design.md` en `51fbd2a1`).

## Cómo se verifica

La nomenclatura de archivos, la unicidad del H1, los enlaces relativos rotos y los marcadores de pendiente son mecánicos: los verifica el CI. La claridad de una explicación, la pertinencia de un comentario y la honestidad de un mensaje de commit no lo son, y quedan para la revisión — que es donde deben estar.

**El contrato de las specs lo verifica `vc/tools/validate-specs.mjs`** (paso «Contrato de las specs» del job `governance` y `G9b` en la batería local). Valida la cabecera, el vocabulario de `status` y `level`, que `decides` y `plan` estén cuando el estado los exige y que los `R-n` no se repitan. Una spec anterior al 2026-10-19 sin cabecera solo se cuenta.

**La nomenclatura de datos y rutas la verifica `vc/tools/validate-data-naming.mjs`** (paso «Nomenclatura de datos y rutas» del job `governance` y `G4d` en la batería local). Lee las colecciones de todo `firestore.rules`, las claves y los `z.enum` de los esquemas zod en `vc/<proyecto>/packages/schema/src/` y los archivos de `apps/<app>/public/` y `hosting/public/`. Sobre los nombres que lee, la forma se verifica entera; el idioma, con un léxico de palabras en castellano que atrapa la reincidencia del vocabulario conocido y crece cuando la revisión encuentra una que se escapó. Es una lectura por patrones, no un parser, y no ve: un `z.enum` armado con una constante, una clave cuyo valor es un esquema en minúscula (`key: baseSchema`), esquemas fuera de `packages/schema/src/`, rutas del enrutador de una app y campos que solo existen como tipos de TypeScript. Eso sigue en revisión, contra la tabla de [Nomenclatura de datos y rutas](#nomenclatura-de-datos-y-rutas). La deuda se tolera **por nombre dentro de su proyecto**, no por archivo: un nombre declarado de Melquisedec pasa en cualquier archivo de Melquisedec hasta que el contrato v2 lo quite del registro, y un nombre nuevo que incumple falla.

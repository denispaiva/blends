# Integración y despliegue — estrategia del portfolio

Cómo se verifica, se integra y se despliega el trabajo en cualquier repositorio del portfolio. Instancia la [invariante 42](role-matrix.md) del estándar de gobernanza y se aplica hoy en Blends; Pinot Index y Teyst la adoptan al activar su ciclo.

## Principio

La verificación automática es lo que convierte una **invariante declarada** en una **garantía**. Un documento que dice «no se commitean secretos» describe una intención; un hook que rechaza el commit describe un hecho.

La disciplina no escala. Funciona mientras el trabajo es de una persona atenta en una sesión corta, y se rompe en la sexta sincronización manual de la misma versión, en el agente que no leyó el estándar, o en el viernes a las once de la noche. Lo que se puede verificar en una máquina se verifica en una máquina; lo que no —la claridad de una explicación, la honestidad de un mensaje de commit— queda para la revisión humana, que es donde debe estar.

De ahí la regla de reparto: **si un check se puede escribir, se escribe**. Agregar un párrafo al estándar sin agregar su verificación es documentar un deseo.

## Biblioteca compartida

Los workflows reutilizables viven en `blends/.github/workflows/` y se consumen **por tag**, no por rama:

```yaml
jobs:
  ci:
    uses: denispaiva/blends/.github/workflows/ci-node.yml@ci-v2
```

El repositorio que **aloja** la biblioteca es la única excepción: se consume a sí mismo por ruta local (`./.github/workflows/ci-node.yml`), sin tag. Fijar un tag ahí significaría que un cambio del pipeline no se prueba hasta después de publicarse, que es exactamente al revés de lo que se busca. Todo repositorio consumidor sí fija tag.

Los tags del pipeline (`ci-v1`, `ci-v2`, …) son independientes de la versión del framework: el pipeline evoluciona a su propio ritmo y un consumidor sube de versión cuando decide, no cuando alguien pushea a `main`.

Copiar plantillas entre repositorios es la alternativa que se descartó: garantiza deriva, porque nadie propaga una corrección a cinco copias.

Los repositorios privados no pueden consumir workflows de otro repositorio privado hasta que el dueño lo habilite. Se aplica por API, no por la interfaz web:

```sh
gh api -X PUT repos/OWNER/REPO/actions/permissions/access -f access_level=user
```

Se verifica con la misma ruta en `GET`: debe devolver `{"access_level":"user"}`.

Blends consume su propia biblioteca. Es la única forma de que el pipeline no quede sin probar: cada cambio al workflow base se ejecuta contra el repositorio que lo aloja antes de que ningún consumidor lo vea.

## Las seis capas

Todo repositorio activo verifica, como mínimo, estas seis capas. El workflow base `ci-node.yml` las instancia para el stack Node; otro stack instancia las mismas capas con sus herramientas.

| Capa                          | Qué protege                                                             | Implementación en `ci-node.yml`                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ----------------------------- | ----------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Instalación reproducible      | Que el entorno del CI sea el del desarrollo                             | `actions/setup-node` leyendo `node-version-file` (`.nvmrc` por defecto), seguido de instalación de dependencias con el gestor detectado del archivo de bloqueo, en modo bloqueado                                                                                                                                                                                                                                                                        |
| Análisis estático             | Errores de sintaxis y código que no carga                               | `node --check` sobre `.mjs` y `.js` versionados, con exclusiones declaradas en `static-analysis-exclude` (rutas exactas, no patrones ni directorios); `static-analysis-command` reemplaza la capa entera                                                                                                                                                                                                                                                 |
| Lint                          | Estilo y patrones que el análisis estático no cubre                     | Vacío por defecto: la capa se omite. `lint-command` la activa, y su fallo rompe el paso                                                                                                                                                                                                                                                                                                                                                                  |
| Pruebas                       | Las regresiones que el autor no anticipó                                | Sin `test-command`, descubre y corre `*.test.mjs`/`*.test.js` con `node --test`; con `require-tests: true`, la ausencia de pruebas propias falla. Con `test-command` declarado, ese comando reemplaza el descubrimiento entero y `require-tests` deja de aplicar                                                                                                                                                                                         |
| Build con detección de deriva | Artefactos generados que quedaron atrás de su fuente                    | Ejecuta `build-command` y falla si `git status --porcelain` ve cambios en `drift-paths`. Sin `build-command`, la capa se omite si tampoco hay `drift-paths`, y falla si los hay: la deriva no significa nada sin un build que la produzca. Valida además sus propias rutas: una ruta ignorada por git, inexistente, o un `drift-paths` que no deja ninguna ruta utilizable tras separarlo por espacios, hace fallar el job en vez de aprobar en silencio |
| Escaneo de secretos           | Credenciales en la historia, aunque los hooks locales se hayan salteado | Compuerta Betterleaks 2.0.0-rc.1 (binario fijado por SHA-256 y cosign) sobre la historia completa (`fetch-depth: 0`), respetando `.betterleaksignore`; gitleaks corre en sombra y es el rollback ([`secrets.md`](secrets.md#compuerta-y-sombra))                                                                                                                                                                                                       |

Esa validación de rutas responde a una regla general que este pipeline aprendió tres veces por las malas: **una capa de verificación que no puede verificar nada debe fallar, no pasar.** Las tres veces que mintió en verde fue por un pathspec vacío que se leyó como ausencia de problema: una ruta ignorada por git, una ruta inexistente, y —la más reciente— un `drift-paths` de solo espacios que dejaba el array de rutas sin un solo elemento, sin que nada lo dijera. La misma regla gobierna `static-analysis-exclude`: una exclusión que no coincide con ningún archivo verificado también falla, nombrándola.

### Contrato de inputs

Todos los defaults preservan el comportamiento de `ci-v1`: un consumidor que no declare nada obtiene el pipeline anterior.

| Input                     | Default  | Efecto                                                                                                     |
| ------------------------- | -------- | ---------------------------------------------------------------------------------------------------------- |
| `node-version-file`       | `.nvmrc` | Archivo que fija la línea de Node                                                                          |
| `package-manager`         | `auto`   | Detecta por archivo de bloqueo; `npm`, `pnpm`, `yarn` fuerzan; `none` omite                                |
| `static-analysis-command` | `""`     | Vacío corre el `node --check` propio; con valor, lo reemplaza                                              |
| `static-analysis-exclude` | `""`     | Rutas exactas excluidas de la verificación propia, separadas por espacios (no son patrones ni directorios) |
| `lint-command`            | `""`     | Vacío omite la capa                                                                                        |
| `test-command`            | `""`     | Vacío corre el descubrimiento propio; con valor, lo reemplaza                                              |
| `require-tests`           | `false`  | Solo aplica al descubrimiento propio                                                                       |
| `build-command`           | `""`     | Comando que regenera artefactos versionados                                                                |
| `drift-paths`             | `""`     | Rutas que no deben cambiar tras el build, separadas por espacios                                           |

Un consumidor sin `.nvmrc` en la raíz ve fallar la preparación de Node con un mensaje que no menciona el input: `node-version-file` es el punto de override, para el consumidor que fija la línea de Node en otro archivo o con otro nombre.

`node --check` solo es efectivo sobre `.mjs`: en un `.js` con sintaxis de módulo se traga el error y sale en cero. Un proyecto TypeScript o ESM declara `static-analysis-command` y reemplaza esa verificación entera.

El descubrimiento propio de pruebas solo reconoce `*.test.mjs` y `*.test.js` corridas con `node --test`. Un proyecto con pruebas en `.ts` las declara por comando.

`ci-v2` no configura caché de dependencias por su cuenta. `actions/setup-node` habilita caché de npm automáticamente cuando el `package.json` del consumidor declara npm en `packageManager` (o en `devEngines.packageManager`), y ese comportamiento por defecto de la acción no se desactiva. Para pnpm y yarn no hay caché: activarlo exigiría tener el gestor en el `PATH` antes de fijar la versión de Node, y ese orden deja los shims de corepack colgando del Node que se reemplaza después. Se reconsidera cuando el pipeline duela.

Probar el workflow contra un consumidor hipotético confirma que compila; no confirma que instale nada real. Por eso el criterio de salida de `ci-v2` exige además una corrida contra un consumidor externo real: contra un proyecto Expo con pnpm, TypeScript y vitest, confirmó lo que ningún sustituto podía probar por sí solo — que corepack fija la versión declarada en el campo `packageManager` del consumidor, y que la instalación en modo bloqueado opera contra un archivo de bloqueo de verdad.

Dos precisiones que la práctica impuso:

- **La deriva se detecta con `git status`, no con `git diff`.** Un artefacto nuevo y sin trackear no aparece en el diff, y esa es exactamente la deriva que más cuesta ver.
- **Los inputs del workflow viajan por `env:` y se leen como variables de shell entrecomilladas.** Interpolar un input directamente en un bloque `run:` permite inyección de comandos desde el repositorio consumidor. Actions además evalúa expresiones en todo el bloque `run:`, comentarios incluidos: escribir la secuencia de apertura de expresión ahí adentro rompe la compilación del workflow aunque esté comentada.

El escaneo de secretos es la última capa por una razón: es la única que mira la **historia** y no el estado actual. Un secreto borrado en el commit siguiente sigue siendo un secreto publicado, y por eso los hooks locales —previo al commit y previo al push— y este check son tres controles distintos y los tres obligatorios: los hooks miran lo que está por entrar a una máquina y por salir de ella, este mira lo que ya quedó escrito.

## Checks de dominio

Las seis capas son el piso, no el techo. Cada proyecto agrega los checks que verifican **sus** invariantes, en un job propio y separado del base.

En Blends son nueve validadores, todos sin dependencias externas. La tabla decía cinco hasta el 2026-08-23 mientras el job `governance` corría ocho: los tres que faltaban se agregan acá junto con el noveno.

| Validador                                      | Qué verifica                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `roadmap/build.mjs`                            | El contrato del dashboard: las fuentes validan contra el schema y los enlaces entre tooling y billing resuelven. Se corre con `--check` para solo verificar, sin regenerar `data.js`. Que el `data.js` publicado coincida con las fuentes lo garantiza la capa de deriva, no este check                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `vc/tools/validate-versions.mjs`               | Coherencia de la versión del framework en los ocho puntos que la declaran, **y que la serie de versiones de los dos CHANGELOG sea monótona**: se leen de lo más nuevo a lo más viejo, y con varias sesiones eligiendo número al ramificar el orden real lo decide el merge. Sin esto, un `main` que fuera 1.56 → 1.58 → 1.57 pasaba en verde                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `vc/tools/validate-billing-ledger.mjs`         | Compromisos, actuals, fuentes y allocations del ledger FinOps                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `vc/tools/validate-skills.mjs`                 | La política de admisión de skills: procedencia, evidencia y responsable                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `vc/tools/validate-docs.mjs`                   | Lo mecánico de las convenciones de markdown: un solo H1, enlaces relativos que resuelven, ausencia de marcadores de pendiente y nombres en kebab-case                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `vc/tools/validate-tooling.mjs`                | Fiabilidad del inventario de tooling: evaluación, evidencia y estado de cada herramienta declarada                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `vc/tools/check-lifecycle.mjs`                 | EOL de tecnologías activas, deadlines externos y `modelBaseline` de skills e instrucciones contra el binding vigente                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `vc/tools/aggregate-tooling-usage.mjs --check` | Forma de los records de uso de tooling (`vc/tooling/usage/records/*.json`, uno por rama) y consistencia del mapa de identidad; nunca los ids, que se resuelven al agregar                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `vc/tools/check-branch-scope.mjs`              | Que el diff de una rama caiga dentro del alcance que su nombre declara (`vc/tooling/branch-scopes.json`). Corre solo en PRs, como **paso final del job `governance`** — que por eso clona con `fetch-depth: 0`, la historia que el diff contra `main` necesita. Sigue en modo advertencia, ahora con `continue-on-error` **a nivel del paso**: falla sin tumbar `governance`. Como `continue-on-error` hace que la API reporte el paso con `conclusion: success`, el paso emite un **`::warning::`** con el archivo culpable antes de salir con `exit 1` — la advertencia se ve en el resumen de la corrida sin abrir el log, y el `exit 1` es lo que hace que quitar el `continue-on-error` baste para promover el check. Sus observaciones, en `docs/tooling/branch-scope-observations.md` (precursor, `vc/docs/tooling/branch-scope-observations.md` en `51fbd2a1`) |

El criterio para agregar un check de dominio es directo: **una invariante que ya se violó una vez merece un validador**. `validate-versions.mjs` existe porque la versión del framework se sincronizó a mano seis veces en una sola sesión.

Un check de dominio que no está en el estándar no es opcional: es incompleto. Cuando el proyecto declare una gobernanza multiagente activa, `tools/validate-governance.mjs` se suma a esta lista.

## Integración

- **Ramas cortas y enfocadas.** `feat/`, `fix/`, `chore/` o `experiment/` con nombre en `kebab-case`, según las [convenciones](conventions.md).
- **`main` se integra por pull request con CI verde.** Nunca por push directo, ni siquiera para un cambio de una línea.
- **Un PR con CI en rojo no se mergea.** Se arregla la causa y se vuelve a pushear; desactivar el check no es arreglarlo.
- **Antes de sincronizar con la nube se consulta el modo de escasez.** Abrir o actualizar un PR, `rerun`, `workflow_dispatch` y merge están gobernados por la política de escasez de cupos de GitHub (precursor, `vc/docs/standards/github-scarcity.md` en `51fbd2a1`): el cupo de Actions es de la cuenta, no del repositorio, y cuando se agota la plataforma no degrada, bloquea. Commitear y pushear ramas sin PR está permitido en todos los modos.
- **La entrega por defecto no es un PR: es una rama en la cola.** Una sesión termina con `npm run entregar` (`vc/tools/deliver.mjs`), que deja la rama pusheada, la batería en verde y la entrada en la cola; el hook `pre-push` no deja publicar sin batería; **no abre PR —ni draft— hasta que le toca el turno**. Un draft no es gratis: dispara los mismos workflows que un PR listo y cuesta lo mismo, así que "dejarlo en draft" nunca fue un ahorro. Los órdenes de sync 1 y 2 son la excepción, abajo.
- **La integración es por tren, no de a un PR por vez.** En cada ventana de integración las ramas de la cola se integran en orden sobre una rama `tren/<fecha>` nacida de `main`, y el tren viaja como **un solo PR** que se verifica entero y se mergea entero. La cadencia es elástica y la fija el sobre diario del modo de escasez (`github-scarcity.md` (precursor, `vc/docs/standards/github-scarcity.md` en `51fbd2a1`), sección "El sobre diario y la cadencia elástica"): el tren sale cuando hay vagones y el sobre tiene saldo, con un disparo cada 2 horas.
- **Exentos del tren: los órdenes de sync 1 y 2.** Un fix que destraba el CI, la gobernanza o un secreto expuesto (orden 1) abre PR individual inmediato: hacerlo esperar la ventana lo convierte en el bloqueo que venía a sacar, y ningún tren posterior va a salir verde mientras la causa siga en `main`. Un PR ya revisado al que solo le falta CI (orden 2) también abre solo, porque meterlo en un tren volvería a poner su revisión en juego junto con la de los demás vagones.
- **Los órdenes 5 y 6 se agrupan por dominio antes de encolar, siempre.** Docs, formato y refactors sin cambio funcional no se encolan de a una rama por cambio: se consolidan por dominio —el mismo que `check-branch-scope` (precursor, `vc/docs/tooling/branch-scope-observations.md` en `51fbd2a1`) ya obliga a respetar— y se encola una entrada por dominio. Tres ramas de docs del mismo dominio pagan tres veces el piso por el trabajo de una.

### El tren de integración

El armado es local y verifica a cada paso. El orden de los vagones es el de la cola: primero `abarata_pipeline`, después `orden_sync` ascendente, y a igual orden, de la rama más chica a la más grande.

```sh
node vc/tools/train.mjs --dry-run   # ensayo: qué vagones abordarían, sin tocar nada
node vc/tools/train.mjs             # armado real: publica la rama, abre el PR y marca flagged lo que no subió
```

`node vc/tools/train.mjs` (`npm run train`) reemplaza los pasos 4 y 5 de `/scarcity` (`.claude/commands/scarcity.md`): arma `tren/<fecha>` vagón por vagón en un worktree propio, corre la batería sobre el acumulado después de cada uno y deshace el merge de cualquier vagón que la rompa. Sale `0` con el tren armado (o con nada elegible, y el motivo), `1` si la batería quedó roja al rehacerla sobre el HEAD actual, `2` por mal uso y `3` cuando el comando necesita un juicio que no puede tomar solo —modo de escasez vencido, un tren de esta ventana ya en vuelo, un armado anterior sin resolver, o ningún vagón abordó (todos los seleccionados fallaron el armado o la batería, o quedaron retenidos)—. `--local` arma en un worktree de inspección sin publicar nada —sin push, sin PR, sin escritura en la cola— para cuando `--dry-run` no alcanza y hace falta ver el reporte antes de exponerlo; se descarta con `--drop <fecha>`.

**El código de los vagones corre sin credenciales** (decisión del PO del 2026-10-27, opción A de `BL-20261027-la-bateria-del-tren-corre-codigo-de-los`). Durante el armado, la batería sobre el acumulado, `roadmap/build.mjs` y los `pnpm install` del worktree del tren ejecutan pruebas, validadores y scripts de ciclo de vida que traen los vagones. Corren con un entorno armado desde una lista blanca —no heredado—, con `HOME` en la carpeta de cachés del usuario (`~/Library/Caches/blends-train-sandbox/<hash del repo>/home`, fuera del repositorio), donde no hay sesión de `gh`, y con un `TMPDIR` nuevo por corrida en el temporal del sistema; git corre sin config de sistema ni global y con un techo de búsqueda, y ssh sin `~/.ssh/config`, sin agente y sin claves (`vc/tools/lib/sandbox-env.mjs` (precursor, `vc/tools/lib/sandbox-env.mjs` en `51fbd2a1`)). El tren corre git con el entorno completo después de cada uno de esos pasos, así que antes y después compara la huella de lo que git ejecuta —la config efectiva que lee el git del tren (sistema, global, XDG, includes, repo y worktree, salvo `branch.*` y `remote.*.gh-resolved`), el puntero `.git` y los archivos del gitdir de ese worktree, los hooks, la carpeta de `core.hooksPath` y los atributos— y, si cambió, o si aparece una config de `gh` en el sandbox, sale `3` sin correr ningún git más y deja el worktree para que una persona lo revise. La huella es una foto: por eso, además, desde que se abre el sandbox todo git del tren corre con `core.fsmonitor=false` y con el `core.hooksPath` y los `credential.helper` que había al empezar, fijados por `GIT_CONFIG_COUNT`. El sandbox persiste entre trenes por las cachés de npm y de los emuladores de Firebase (el store de pnpm es el de `PNPM_HOME`): el primer tren después del cambio las vuelve a bajar y necesita red. Qué no cierra, porque el hijo corre con el mismo usuario del sistema: leer el entorno inicial de los procesos padre con `ps -E`, consultar el llavero por su ruta absoluta, leer o escribir cualquier archivo del usuario y dejar un proceso vivo que escriba después de la foto (un filtro `filter.*`, un helper por URL o un hook reescrito en la carpeta de confianza todavía podrían correr); y el `validate-versions.mjs` del acumulado se importa dentro del propio proceso del tren. La frontera real es otro usuario o un contenedor (opción B, abierta).

**Numera al abordar.** Una rama que trae un bump de versión lo pre-declara contra la base en la que se ramificó, pero el número que termina en `main` lo asigna el tren, en el orden real de embarque —no el orden en que se encoló—: si dos ramas reclamaban el mismo número, la que aborda primero se lo queda y la siguiente recibe el próximo libre. Es la forma de no depender de que ninguna sesión paralela adivine bien el número final.

**Qué conflictos resuelve el tren y cuáles no.** Los que no dicen nada del contenido del vagón los resuelve solo, y el resto sigue rechazándose con el archivo y el motivo, sin perder nada en silencio:

- **Los nueve puntos de versión y los changelogs**, como siempre: se toma el número del acumulado y se numera al abordar.
- **`SCOPE-EXCEPTION.md`.** Es una declaración por rama que el tren lee de `origin/<rama>`, nunca del acumulado, así que dos vagones que lo escriben chocan siempre y el choque no dice nada de ninguno: queda el del acumulado.
- **Un punto de versión con otros cambios del vagón en el mismo archivo.** Las tres versiones del archivo (base, acumulado y vagón) se llevan al mismo número antes del merge de tres vías y después se repone el del acumulado. Con eso ni un cambio contiguo ni uno en la misma línea (el README dice el número del framework y el del schema a la vez) chocan con el bump. Lo mismo vale para el `updatedAt` de un `.roadmap.json`, que cada vagón que lo toca cambia con su fecha: queda la más reciente. Si lo propio del vagón choca de verdad con algo del acumulado, es un conflicto de contenido y el vagón no sube.
- **Un vagón apilado sobre otro que abordó renumerado** declara la versión de **su** punta, no la del árbol ya mergeado: sin cambio propio de versión sube sin bump, y no retrocede ni suma uno que nadie pidió.

**Los puntos de versión son los del árbol que se arma.** Un vagón puede agregar un punto de versión o mudar uno de lugar —el motor FinOps movió el schema del ledger a `vc/brix/`—, así que el tren no usa la lista de su propio checkout: lee `SOURCES` del `validate-versions.mjs` del acumulado y del de la punta de cada vagón (agregándole el `export` si falta, porque los vagones anteriores a la cadena de `train-script` declaran `const SOURCES` a secas), clasifica los conflictos con la unión de las dos listas y lee la base de un archivo movido desde su nombre viejo, sea el vagón o el acumulado quien lo movió. Un punto sin `replace` se reescribe derivándolo de su `extract`: se prueba reemplazar una aparición del número por vez y se acepta la que hace que `extract` lea el valor nuevo, de modo que otro campo con el mismo número no se toca. Al final todos los puntos del árbol mergeado quedan en el número que le tocó al vagón, partiendo del del acumulado o del propio del vagón (un punto agregado trae el suyo). Un vagón que cambia el **primer** punto, `roadmap/build.mjs`, que es la fuente de verdad de la versión, no sube y el reporte dice por qué.

**Un vagón con dos padres ya abordados.** Una rama que absorbió a otra (`train-script` absorbió a FinOps) tiene dos merge-bases con el acumulado, y contra uno solo lo que el vagón "agregó" incluye lo que trajo el otro padre. El tren calcula el alcance contra cada base y se queda con lo que difiere de todas, no cuenta como alcance propio los archivos de versión que el merge combinó, y en los changelogs deja las entradas que el acumulado ya tiene (comparadas por línea, porque el tren renumeró los encabezados y otra rama pudo haberles agregado una viñeta). Si el vagón trae una entrada propia, no la pierde: se rechaza y el reporte dice cuál.

**Renumerar no amplía el alcance.** Subir de versión toca todos los puntos, incluidos archivos que la regla del vagón no admite (los ledgers por proyecto). Un archivo cuyo único cambio frente a la base es el número de versión no cuenta para el alcance; si en él cambia además otra cosa, sigue contando.

Lo que el tren **no** resuelve, y es lo correcto: dos vagones que editan las mismas líneas de un archivo que no es de versión (por ejemplo `validate-versions.mjs` o `validate-tooling.mjs`, o secciones agregadas al final de `evaluations.md`). Es un conflicto de contenido entre ramas independientes: el vagón que queda atrás no sube, el reporte nombra el archivo, y se rebasa sobre el `main` que dejó el tren y se re-entrega. Desde el ensayo, ese choque lo descubre `enqueue` en el momento de encolar y no el tren (ver [Ensayo del tren](#ensayo-del-tren)): el vagón queda retenido como `preflight-conflict` con el mismo motivo y el resto del tren sale sin él.

**El alcance se verifica por vagón contra su propio prefijo, no contra `tren/*`.** Cada vagón evalúa su propio diff de merge-base —contra la rama de la que nació, no contra `tren/<fecha>`— frente a `branch-scopes.json` (precursor, `vc/docs/tooling/branch-scope-observations.md` en `51fbd2a1`), y el resultado queda en la columna "Alcance" del reporte (`outputs/tren-<fecha>.md`). Cierra el ítem que había quedado abierto en el ciclo de los tres trenes: `tren/*` no entra en la taxonomía de alcance, y no hace falta que entre —lo que se mide es el trabajo de cada vagón contra su propio prefijo declarado, no el vehículo transitorio que los lleva a `main`.

**`npm run verify` es la batería.** Reemplaza a la instrucción de "correr los
validadores", que no nombraba ninguno y dejaba a cada sesión decidir qué
contaba como completo. Transcribe los pasos de `blends-ci.yml` y `ci-node.yml`
con su directorio y sus requisitos (`vc/tools/verify.mjs` (precursor, `vc/tools/verify.mjs` en `51fbd2a1`)),
y devuelve **tres estados con tres códigos de salida distintos**:

| Estado       | Código | Qué significa                                                |
| ------------ | ------ | ------------------------------------------------------------ |
| `passed`     | 0      | Todas las comprobaciones aplicables se ejecutaron y pasaron. |
| `failed`     | 1      | Al menos una falló. Gana sobre cualquier incompletitud.      |
| `incomplete` | 2      | Algo no se pudo verificar en condiciones equivalentes.       |

**Solo el 0 habilita a decir que el vagón está verde.** Un `incomplete` no es un
verde con asterisco: significa que faltó una herramienta, que el árbol de
dependencias no se pudo inspeccionar, que el entorno difiere de CI en algo que
cambia el resultado, o que los workflows cambiaron y el mapeo de `verify` ya no
los describe. El comando dice cuál de esas cosas pasó y qué falta para
completarlo.

**`verify` no sustituye a CI y no lo dice como formalidad.** "Batería local
completa" es el exit 0 sobre un `HEAD`; "CI aprobado" son los cuatro checks
requeridos en verde sobre ese mismo SHA. El escaneo de secretos es el caso
claro: `verify` lo reproduce, pero el control de que se escaneó _lo publicado_,
en una máquina que no es la de quien escribió el commit, sigue siendo de CI
(ver la capa 4 de este documento). Al citar la batería en un PR se pega el
registro (`.verify-report.json`) con su `HEAD`, y quien mergea compara ese
`HEAD` con el del PR.

**La batería se re-corre sobre el acumulado después de cada vagón, no una vez al final.** Una rama que pasa sola puede romper el acumulado —dos que tocan el mismo log append-only, una versión que colisiona con otra, un enlace relativo que la otra borró— y el punto del tren es que eso se descubra en local, donde cuesta cero, en vez de en la nube. Si el acumulado falla, ese vagón **no sube**: se deshace su merge, vuelve a la cola con el hallazgo anotado en su entrada, y el tren sigue con el siguiente. Un vagón que no sube no cancela el tren.

**Si el tren sale rojo en la nube** —pasó en local y falla en CI—, se saca la rama culpable y se rearma. **La reproducción se hace primero en local**: el run dice qué job falló y la sesión lo corre en su worktree sobre el acumulado. **La bisección en la nube es el último recurso**, porque cada intento cuesta un run entero del tren, que es exactamente el gasto que el tren viene a evitar. La culpable vuelve a la cola con el hallazgo anotado; el tren rearmado, sin ella, viaja igual.

**Por qué el tren, medido sobre 2026-09-01 → 2026-09-05.** Cinco días del ciclo 2026-09: **76 PRs abiertos** (#136 (PR 136 del precursor)→#211 (PR 211 del precursor)), **243 corridas**, **1143 jobs**, **1813 minutos facturados**. El presupuesto sostenible del pool se calcula como pool mensual ÷ precio por minuto ÷ 30 días; en la instancia que originó la regla daba **100 min/día**. La quema real fue de **363 min/día —3,6× el presupuesto— y acelerando**: 89, 466, 282, 356 y 620 minutos por jornada. Al ritmo del quinto día, un pool de treinta se agota en cinco.

**El 41,9 % de lo facturado no es cómputo, es redondeo.** Esos 1813 minutos corresponden a **1053,8 minutos de trabajo real**: Actions redondea hacia arriba **por job**, y la diferencia —**759 minutos**— es piso puro. Está concentrada en los jobs más cortos: `governance` promedia **9,8 s** y paga un minuto; `branch-scope`, **5,4 s**, y paga un minuto. Los dos juntos suman **57 minutos de trabajo real y 450 de factura**. De ahí sale la regla: **el costo de una entrega no es proporcional al tamaño del cambio, sino a la cantidad de veces que se abre una corrida.** La corrida más barata del período gastó 92 segundos de cómputo repartidos en cinco jobs y facturó cinco minutos; 71 de las 225 corridas de `Blends CI` (32 %) pagaron exactamente ese piso. Con el pliegue de #200 (PR 200 del precursor) el piso baja a cuatro minutos, pero **sigue siendo por corrida**: la única forma de pagarlo una vez en lugar de N es abrir una corrida en lugar de N.

**Y el régimen anterior multiplicaba las corridas por diseño.** Con `strict: true`, cada merge obliga a un `update-branch` de los demás PRs abiertos, y cada `update-branch` es una corrida nueva. Medido: **72 de las 243 corridas tuvieron como `head_commit` un merge de `main` en la rama —554 minutos facturados, el 30,6 % del gasto del período— sin producir una sola línea de trabajo nuevo.** El tren elimina ese término de raíz: nace de `main`, viaja solo y mergea una vez; no hay otros PRs abiertos a los que re-sincronizar.

**El método, porque un número sin método es lo que hizo falsa la cláusula de #129 (PR 129 del precursor) en `github-scarcity.md` (precursor, `vc/docs/standards/github-scarcity.md` en `51fbd2a1`).** Las corridas salen de `GET /actions/runs?created=>=<cycleStart>` y los jobs de `GET /actions/runs/{id}/jobs`; la factura se reconstruye como `ceil(duración_job / 60 s)` con mínimo de un minuto por job. **El endpoint `/timing` no sirve en este plan: devuelve `total_ms: 0` en las 243 corridas**, así que una medición que lo use va a reportar cero y creerlo. Validación: contra el mismo `measuredAt` del monitor (2026-09-05T12:44Z) la reconstrucción da 1730 minutos contra 1660 medidos, **+4,2 %**, explicable por cuatro corridas canceladas y un `pendingId` del incremental.

_Estimación, no medición:_ si el trabajo de esos cinco días hubiera viajado en un tren diario, las 225 corridas de `Blends CI` habrían sido cinco trenes más los PRs exentos de orden 1 y 2. Aun suponiendo, de forma deliberadamente conservadora, que la mitad de los 76 PRs fueran exentos, el orden de magnitud cae de 1791 a unos 360 minutos. **Es una estimación con sus supuestos a la vista, no un dato: la cifra real la dará el primer tren con cola poblada, y se registra al mergearlo.**

### Auto-merge del tren

Tras el armado, la tarea programada corre `node vc/tools/train-gate.mjs` (`--pr N` para elegir tren). Evalúa ocho compuertas sobre el PR —CI verde y `CLEAN`, modo `normal`, ritmo dentro del sobre diario, sin `SCOPE-EXCEPTION.md` en el diff, sin archivos del plano de control (`vc/tooling/control-plane.json`, leída del `main` contra el que evalúa), sin vagones de orden 1–2, a lo sumo 6 vagones, ningún otro tren abierto— con la evidencia completa (todas las páginas de archivos y commits, formación verificable desde los commits `merge: vagón i/N`) y registra `mergearía / no mergearía / indeterminado` con la causa en un comentario propio del PR, con historial. Cada viaje tiene nombre propio (`tren/AAAA-MM-DD`, `-2`, `-3`…). Sin `BLENDS_TRAIN_AUTO_MERGE=1` corre en sombra y no mergea; con la variable —que el PO activó el 2026-10-25 al cierre de la semana en sombra— hace `gh pr merge --merge --match-head-commit <sha>` en cuanto las siete compuertas pasan y la decisión quedó registrada. El gate no corre el ensayo (decisión del PO del 2026-10-19): decide solo con datos del propio PR. Lo que espera a una persona —tren retenido, orden 1–2 sin PR— se avisa con un issue de `github-actions[bot]` vía `train-notice.yml`, solo cuando cambia de estado. Las compuertas, su porqué, la activación, el aviso y la cadencia elástica están en `github-scarcity.md` (precursor, `vc/docs/standards/github-scarcity.md` en `51fbd2a1`). A pedido del PO en el chat, un agente mergea el tren con `node vc/tools/train-merge.mjs --pr N --po-approved "<cita>"`, que vuelve a correr las compuertas, acota el merge a `tren/*` → `main` sobre el SHA evaluado y registra la cita del PO y quién lo ejecutó (ver `github-scarcity.md` (precursor, `vc/docs/standards/github-scarcity.md` en `51fbd2a1`), párrafo «Merge a pedido del PO»).

### Ensayo del tren

El conflicto que recién aparece el día del tren llega tarde: quien lo provocó ya no está y el tren sale sin esa rama. El ensayo mueve el descubrimiento al momento de encolar. **Definición de éxito:** después de encolar una rama no existe ningún conflicto conocido que recién pueda descubrirse en T-0. Dos corolarios, y los dos son un defecto del sistema, no de la rama:

- Un vagón `ready` que en T-0 falla con los mismos `main`, SHA y formación es un **bug del preflight**. El reporte del tren lo dice con esas palabras (`BUG DEL PREFLIGHT`).
- Un `main`, una rama o una entrada anterior que cambió sin que la validación pasara a `stale` es un **bug del modelo de estado** (`BUG DEL MODELO DE ESTADO`).

```sh
node vc/tools/train.mjs --rehearse                    # ensaya la cola entera
node vc/tools/train.mjs --rehearse --target <rama>    # igual, y el exit lo decide esa rama
node vc/tools/queue.mjs enqueue …                     # ensaya solo, después de escribir la entrada
```

**Qué es un ensayo.** `main` más las entradas de la cola, en el orden del tren, armadas con el mismo `boardWagon` que usa el tren, en un worktree descartable, **sin batería, sin escasez y sin publicar nada**. Corre en los cuatro modos: no consume minutos de Actions, así que ni `congelado` ni `agotado` lo impiden. `boardWagon` no sabe si corre en un ensayo o en un tren: lo que cambia entre uno y otro (la batería, el SHA que se mergea) entra por `ctx`. Siempre arma la formación completa: la punta predicha de la cuarta entrada exige haber abordado las tres primeras. El resultado entra a la cola con **un solo** `queue.mjs ensayo-batch`, y el reporte queda en `outputs/ensayo-<run_id>.md`, también cuando el ensayo falla (con el error y "la cola no se tocó"), para que una corrida fallida no se confunda con una que no corrió.

**Un solo snapshot.** Un ensayo lee `origin/main` y la punta de cada rama de la cola una vez, antes de armar, y no vuelve a consultar origin. Cada vagón se mergea desde el SHA del snapshot, no desde `origin/<rama>`. Un push que llega durante el ensayo **no lo aborta**: el ensayo sigue siendo válido como observación histórica del snapshot en que corrió, pero deja **inmediatamente `stale`** a esa entrada (`stale-branch`) para la próxima lectura. El tren, en cambio, **sí aborta** (más abajo): no publica un armado cuyo snapshot dejó de representar la realidad.

**Los dos ejes.** Que una entrada esté en la cola (`queued`) y qué dice el preflight de ella son cosas distintas. El estado de preflight es uno de:

| Estado                  | Cuándo                                                                                                                                                                   | Se guarda |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------- |
| `untested`              | la entrada no tiene ensayo                                                                                                                                               | no        |
| `stale`                 | tiene ensayo pero venció: `stale-branch` (su SHA o su punta en origin cambió), `stale-formation` (cambió lo que va antes que ella) o `stale-main` (`origin/main` avanzó) | derivado  |
| `predicted-conflict`    | el ensayo no pudo abordarla; `detalle` dice por qué                                                                                                                      | sí        |
| `blocked-by-dependency` | alguna dependencia, transitivamente, está en conflicto, flagged o fuera de la cola                                                                                       | derivado  |
| `ready`                 | el ensayo la abordó y nada de lo que lo produjo cambió                                                                                                                   | sí        |

`stale` y `blocked-by-dependency` se derivan en una función pura (`preflightState`, `vc/tools/lib/queue.mjs`) a partir de un snapshot; ni consulta git ni guarda cachés. Con varias causas manda `stale`, después `blocked-by-dependency`, después `predicted-conflict`. `previousResult` y `secondaryReasons` (por ejemplo "vencido por main; último ensayo: conflicto en X") son **solo observabilidad**: la selección, la retención y el bloqueo dependen exclusivamente del estado principal y del grafo de `depende_de`. Un ciclo en `depende_de` no es un estado sino un error estructural: `enqueue` lo rechaza antes de escribir.

**La formación y la cadena.** El ensayo de una entrada guarda las entradas que iban antes (`formacion`, con su SHA), la punta del worktree antes de abordarla (`base_predicha_sha`) y después (`punta_predicha_sha`). Encolar una rama que ordena antes cambia la formación de las posteriores y las deja `stale-formation` sin que nadie lo declare. La cadena `punta → base` se verifica sobre lo abordado: una entrada en conflicto no mueve la punta y la siguiente se encadena contra la última que sí abordó; una entrada bloqueada no tiene ensayo propio y nunca se encadena contra un vacío. Una cadena rota se denuncia.

**Qué ve `enqueue`.** Después de escribir la entrada y **fuera del lock**, `enqueue` ensaya con `--target <rama>` y sale 0 si quedó `ready`, 1 si el ensayo la predice en conflicto o bloqueada; en los dos casos la entrada queda escrita con su ensayo. Sin origin (`BLENDS_QUEUE_NO_SYNC`) o con la cola fuera del repo (`BLENDS_QUEUE_DIR`) no ensaya. La cola ordena por tamaño dentro de una misma orden, así que una rama que llega después puede quedar **antes** en la formación y romper a una que ya estaba lista: en ese caso `enqueue` también sale 1 y nombra a la entrada rota (`ATENCIÓN: ensayar X dejó en conflicto a Y`). El conflicto se asigna **operativamente** a quien encola, porque la formación existente define su base predicha al momento de encolar. No implica atribución causal sobre el cambio.

**Cómo se resuelve un conflicto predicho.** En la rama, nunca en el worktree de ensayo (es descartable): mergear la **rama real** de la que depende y declarar `--depende-de`, o rebasar sobre `main`; después re-encolar. **Archivo caliente** —dos ramas sin orden declarado que tocan el mismo path— es una **advertencia** del reporte, no un bloqueo: dos cambios al mismo archivo pueden mezclarse limpios, y `depende_de` se declara por dependencia semántica, no por coincidencia de path.

**Exit de `--rehearse`, de `ensayo-batch` y de `enqueue` con ensayo.**

| Exit | Significa                                                                                                                                            |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0    | la formación (o el `--target`) quedó `ready`                                                                                                         |
| 1    | estado operativo no listo: `predicted-conflict`, `blocked-by-dependency`, una entrada retenida o una rama que avanzó desde que se encoló             |
| 2    | datos, contrato o invariante inválidos: un ciclo, un lote de `ensayo-batch` incompleto, o una entrada recién ensayada que quedó `stale` o `untested` |
| 3    | entorno                                                                                                                                              |

Con `--target` el exit lo decide el estado **derivado** de esa rama (`blocked-by-dependency` cuenta aunque no tenga ensayo propio), más las entradas que este ensayo dejó en conflicto; un bug siempre gana. `ensayo-batch` recibe `formation` —todo lo que el ensayo pretendió evaluar— y exige que sea exactamente `results ∪ omitted`: un JSON truncado o un productor que murió a mitad no se confunde con "estas no aplican", y se rechaza entero con exit 2 sin escribir nada.

**Calendario.** El ensayo por entrada hace de red de detección continua; el calendario es la red de seguridad, y se mide contra la **hora programada** del tren (la del disparo que lo arma, cada 2 horas; un tren extraordinario pasa `--scheduled-at` con zona explícita).

| Momento        | Qué pasa                                                                                                                                                                                               |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| entre disparos | ensayo preventivo y publicación de los conflictos (el ensayo por entrada al encolar es la red continua)                                                                                                |
| T-2 h          | **freeze**: la ventana entre dos disparos. Las entradas de orden ≥ 3 encoladas después del corte esperan al disparo siguiente; `--force-cadence` no lo levanta, `--sin-freeze` sí y el reporte lo dice |
| T-0            | el tren solo aborda                                                                                                                                                                                    |

**El tren respeta el ensayo.** Precondición del modo normal: **antes del primer abordaje real, todo candidato está `ready` o retenido.** Si alguno no lo está —sin ensayo, o vencido por `main`, por su rama o por la formación— el tren ensaya la formación completa **primero**, con el mismo snapshot y sin batería, y recién entonces aborda con batería lo que quedó `ready`. No hay revalidación en línea: menos estados cambiando mientras el tren está vivo. Un conflicto predicho o una dependencia bloqueada no aborda y queda retenido con el motivo (no `flagged`: el ensayo ya lo dejó anotado en la cola); si por eso no aborda nadie, el tren sale 3. Una rama que avanzó desde que se encoló se retiene: no se abordan SHAs sin validar.

**El snapshot no puede mutar bajo el tren.** Antes de anotar nada en la cola y otra vez antes de publicar, el tren comprueba con `git ls-remote` que `origin/main` y la punta de cada rama que viajó siguen siendo las del snapshot. Si algo cambió, aborta con exit 3, baja el worktree y no publica ni anota nada; el reporte dice cuál cambió. No se sabe en qué vagón pasó, pero un tren armado sobre un `main` o un SHA que cambió deja de representar la entrada.

### Reciclado

`node vc/tools/recycle.mjs` (`npm run recycle`) cierra el círculo que abre el tren: drena la cola y limpia lo que ya mergeó. No borra nada por su cuenta fuera de lo que cada modo describe; todo lo que no cumple la regla exacta se reporta, nunca se fuerza.

- **`--daily`** borra del remoto **solo** las ramas que están mergeadas por completo en `origin/main`, no están en la cola, no tienen PR abierto y no son `main`, `queue` ni `tren/*` —esas tres, y cualquier rama todavía encolada, quedan siempre protegidas—. Remueve un worktree **únicamente** cuando la rama de ese worktree fue reciclada en la misma corrida —borrada del remoto por esa misma pasada— y el worktree está limpio y su rama no tiene un claim de sesión vivo. **`git worktree remove` borra el directorio de trabajo, no solo la metadata**: lo que había ahí sin commitear no vuelve, y por eso la regla es esa y no "integrada y limpia". Removido el worktree, la rama local se borra como acción aparte (`delete-local-branch`) y solo con la integración verificada contra `origin/main` (`git merge-base --is-ancestor`), nunca por fiat. Los worktrees bajo `.claude/worktrees/` (sesiones de Claude) nunca los retira `recycle`; tampoco `.git-queue/` ni `.git-trains/` —esos los limpia `train.mjs --drop`—. Todo lo demás —un worktree sucio, una rama sin publicar, un `HEAD` desacoplado, una rama con claim vivo— solo se reporta. Un claim **vencido** (TTL de `session-claim`, o su worktree ya ausente) no frena nada: el barrido diario lo libera.
- **`--after-train <PR>`** exige que el PR esté `state: MERGED` —no recicla un tren en vuelo ni uno fallido—. Drena de la cola los vagones que viajaron (los archiva en `archive/` sobre la rama `queue`), libera sus claims, **borra en origin las ramas de los vagones que viajaron** y mide el costo real del tren. Los nombres de rama salen del cuerpo del PR, que es texto: antes de tocar nada se verifica cada uno como nombre de rama y se cruza con `git cherry origin/main origin/<rama>` —vacío ⇒ todos sus parches están en main—; una rama nombrada en el PR que tenga commits fuera de main no se recicla, se reporta. La medición es job por job (`ceil(segundos / 60)`, mínimo un minuto por job que corrió; los jobs `skipped` no se cuentan, GitHub no los factura) sobre el SHA del PR y el commit de merge, y agrega **una sola vez** el bloque `## Costo medido — PR #<N>` a `outputs/tren-<fecha>.md` —es idempotente: reintentarlo no duplica el bloque—. **El drenado también es idempotente:** los vagones del PR se cruzan con la cola viva y los que ya están archivados se saltean con una nota (`ya drenada: <rama>`), así que reintentar la corrida no falla por una entrada que ya no está. Después de drenar registra los `resource-actuals` de M2 (Brix T-13) de las ramas que viajaron, en la rama `queue`, en `resource-actuals/<ciclo>.json` (dato de máquina, sin tren ni Actions); es best-effort: un fallo se reporta como `! M2 resource-actuals: ...` y no cambia ningún código de salida. `--dry-run` solo anuncia lo que produciría. Es el paso 5 de `/scarcity` (`.claude/commands/scarcity.md`) que hasta ahora se corría a mano.
- **`--dry-run`**, en cualquiera de los dos modos, dice qué haría sin hacerlo: **sí** hace `fetch --prune` y **sí** sincroniza la cola para leerla (son lecturas, y sin ellas el plan se decidiría contra una foto vieja), pero no ejecuta ninguna acción —no borra ramas, no remueve worktrees, no libera claims, no drena la cola ni escribe el reporte—. Lo único que un ensayo sí puede devolver es el `4` de diagnóstico: si el cuerpo del PR no tiene la tabla de vagones, lo dice ahí y no espera a la corrida real.
- **Métricas del día.** Los dos modos imprimen un bloque `## Métricas del día`: los minutos facturados del día en **este repositorio**, separados en tren, push a `main`, PRs individuales (`pull_request` cuyo head no es `tren/*`) y el resto, más la proyección del ciclo —el `used` de `GH_SCARCITY_STATUS` más los trenes que todavía entran por cadencia, a costo medido cada uno, contra el `pool`—. **Son métricas por repositorio, no de la cuenta:** la cuenta entera la mide `escasez-monitor`, y de esa medición sale el `used` con el que se proyecta.
- **Códigos de salida.** `0` todo bien; `1` alguna acción falló —las demás se ejecutaron igual, y cada fallo queda impreso con su `stderr`—; `2` no se pudo leer el estado del que depende el plan —`git fetch` de origin, `gh pr view`, `gh pr list`, o mal uso— y no se hizo nada: esas lecturas van todas **antes** del drenado y del bloque de costo, justamente para que el `2` signifique eso; `4` el reciclado se completó pero la cola quedó sin drenar, con la línea `COLA_NO_DRENADA PR #<N>` —porque falló la llamada a `queue.mjs prune`, o porque el cuerpo del PR no tiene la tabla de vagones y entonces no hay ninguna rama que drenar (`: el cuerpo no tiene la tabla de vagones`)—. La cola desincronizada es lo que hay que atender primero, así que el `4` gana sobre el `1` y sobre cualquier salida posterior.

### Salto de la verificación post-merge por árbol idéntico

La corrida que el push a `main` dispara tras el merge del tren vuelve a verificar el mismo árbol que el PR ya verificó en verde: con `strict: true` la rama contenía a `main` al mergear, así que el merge de prueba que corrió el PR y el merge real comparten árbol. El job `tree-gate` (`.github/workflows/blends-ci.yml`) declara `covered=true` solo si se cumplen las cinco condiciones a la vez, todas sobre `p2` (el segundo padre del commit que entró a `main`):

1. el commit es un merge de exactamente dos padres;
2. su árbol es el de `p2`;
3. `GET commits/{p2}/pulls` trae un PR con base `main`, mergeado (`merged_at != null`) y con `merge_commit_sha` igual a este commit;
4. `GET commits/{p2}/check-runs` trae los cuatro checks requeridos de `main` —`base / verify`, `governance`, `lint`, `teyst`—;
5. los cuatro terminaron en `success`: `skipped`, `neutral`, `cancelled` o sin concluir no cubren nada.

Con varios check-runs del mismo nombre se toma el de mayor `id` (el último intento, que es el que GitHub usa para un check requerido) y solo de la app `github-actions`. Si se cumplen las cinco, los jobs `base`, `teyst`, `lint` y `governance` (y `melquisedec`) se saltean. Ante cualquier duda —no es un merge, el árbol difiere, la API falla o devuelve una forma inesperada, falta un check, uno no está en `success`, o el propio script se cae— corre todo, sin excepción.

Cada decisión deja `covered` y `reason` como salidas del job (`covered: validated-pr`, `not-covered: not-two-parent-merge | tree-mismatch | no-associated-merged-pr | merge-sha-mismatch | missing-required-check | required-check-not-success | api-error | internal-error`) y un resumen con los SHA del merge y de `p2`, ambos árboles, el PR detectado y la conclusión de cada check.

**Por qué no se usa `actions/runs`.** La primera versión del gate buscaba la corrida del PR en `actions/runs` y filtraba por `pull_requests[].base.ref`. Ese arreglo viene vacío en las corridas de una rama que ya no existe, y `tren/*` se borra al mergear (`delete_branch_on_merge`): el gate dio `covered=false` en los 11 push a `main` entre 2026-09-26 y 2026-10-05 (119 min facturados post-merge, de los cuales ~108 eran evitables). Los dos endpoints de arriba no dependen de que la rama siga viva.

**Exige la estrategia de merge "Create a merge commit".** Un squash o un rebase producen, en `main`, un commit de un solo padre: `tree-gate` no encuentra segundo padre, sale `covered=false` de inmediato y la verificación post-merge corre completa igual —sin ahorro, pero también sin falso negativo—.

### Esperas sobre CI

Una sesión que espera a que la CI cierre abre un proceso, y ese proceso puede
sobrevivir a lo que esperaba. **Enmendar un commit no mata al esperador del SHA
anterior**: la espera queda apuntando a un objeto que dejó de existir, con una
condición que ya no puede cumplirse, y sigue corriendo. Desde afuera es
indistinguible de trabajo en curso — un contador de tareas la muestra viva y
creciendo, con la misma cara que una tarea legítima.

**Cómo se pregunta, antes de con qué techo se espera.** La consulta se hace
**sobre el objeto**, no sobre una ventana de los últimos N:

```sh
# Bien: le pregunta al objeto
gh api "repos/OWNER/REPO/actions/runs?head_sha=$SHA" --jq '.workflow_runs[0].id'

# Mal: pregunta si el objeto está entre los últimos 20
gh run list --limit 20 --json databaseId,headSha --jq '[.[]|select(.headSha=="'$SHA'")][0].databaseId'
```

Con un repositorio activo, veinte runs cubren pocas horas: _"¿mi SHA está entre
los últimos 20?"_ **no** es _"¿existe un run para mi SHA?"_. Las dos preguntas
coinciden mientras el repositorio esté tranquilo, que es cuando nadie lo nota.

Tres reglas, las tres baratas:

- **Toda espera lleva límite de tiempo, y al vencerlo sale ruidosa.** Dos veces
  la duración típica del run alcanza; lo que no vale es esperar sin techo, porque
  una condición insatisfacible no se distingue de una lenta.
- **Quien enmienda mata la espera previa.** Es responsabilidad de quien reescribe
  el commit, no de quien la abrió: el que enmienda es el único que sabe, en ese
  momento, que el objeto esperado dejó de existir.
- **Cuando una espera vence, el primer objeto a consultar es la plataforma, no el
  repositorio.** Es contraintuitivo y por eso está escrito: el reflejo es buscar
  la causa adentro —el workflow, el trigger, la rama, el permiso— y el 2026-08-26
  **tres sesiones perdieron tiempo haciendo eso** mientras GitHub Actions estaba
  en `major_outage`. Un vistazo a `githubstatus.com` cuesta segundos y descarta
  la mitad del espacio de búsqueda:

  ```sh
  curl -sS https://www.githubstatus.com/api/v2/components.json \
    | python3 -c "import json,sys;[print(c['name'],c['status']) for c in json.load(sys.stdin)['components'] if c['name']=='Actions']"
  ```

Caso fundante: una espera sobre un SHA enmendado del
PR #101 (PR 101 del precursor) siguió corriendo horas
después de que ese PR se mergeara, y la sesión que la había abierto **afirmó que
no existía** — la buscó midiendo procesos por sus marcas de inicio y fin, y un
proceso vivo no tiene marca de fin. Ver el
Ítem 7 (precursor, `vc/docs/standards/framework-debt.md` en `51fbd2a1`) del registro de deuda, instancia 7.

La protección dura de ramas —el bloqueo que la plataforma impone— exige plan pago en repositorios privados. Mientras el portfolio opere en free tier ([Free Tier First](role-matrix.md)), la regla se audita **por convención y por revisión, no por bloqueo de la plataforma**.

Esto tiene una consecuencia que conviene decir en voz alta: la regla es tan fuerte como la disciplina de quien integra, que es justamente lo que este estándar dice que no escala. Se acepta como deuda consciente y acotada —un repositorio, un integrador— y se salda comprando la protección cuando haya más de una persona con permiso de push. Esa compra es una graduación de free tier y exige la aprobación humana que el estándar ya define.

## Verificar contra una URL servida: registrar el artefacto

**Toda verificación contra una URL servida registra el `teyst:commit` que vio, junto al resultado.** Un informe que dice "verificado contra `dev-<proyecto>`" sin el SHA que la URL servía en ese momento **no es atribuible**: afirma algo sobre un artefacto que nadie puede volver a identificar. **Un SHA que no es ancestro de `main` es una bandera** — el artefacto salió de una rama, y lo verificado no es lo que está mergeado.

_(La regla nació el 2026-08-28, de dos hechos: `hosting:dev` acumulaba 39 releases, todos manuales y sin un solo metadato de origen — Firebase guarda fecha y autor, no rama ni commit; y en la descripción de un PR apareció una transcripción fabricada, con el comando de uso arriba y la salida de otro objeto abajo, a la que además se le había quitado el sufijo `-sucio`. Ninguna herramienta podía detectar ninguna de las dos.)_

**Y hay una distinción que evita sobre-invalidar cuando la atribución falla.** No todo lo que se miró en una URL era una afirmación sobre el artefacto servido:

- **Afirmaciones sobre el artefacto** — "el fix funciona", "la consola está limpia", "no hay salto de layout". Sin SHA registrado, se caen: no se sabe qué se probó.
- **Afirmaciones sobre la URL como objeto** — "qué está sirviendo", "hay caché vieja", "devuelve estas cabeceras". Siguen en pie: lo medido era la URL misma, y sigue siendo la misma URL.
- **Juicios sobre un insumo que no vino de la URL** — un veredicto visual tomado sobre assets del repositorio rasterizados localmente no depende del despliegue, aunque el tema sea el mismo. Lo que sí se cae, y es otra cosa, es cualquier afirmación sobre **qué sirve la URL hoy**.

Al invalidar por atribución perdida, clasificar antes de rehacer: rehacer lo que no lo necesita cuesta trabajo y, peor, desgasta la regla.

## Manifiesto de una corrida de migración/recálculo: commiteado en el mismo PR

**Toda corrida real de una herramienta de migración o recálculo contra Firestore deja su manifiesto —qué escribió, cuándo, contra qué proyecto, con qué versión del código— commiteado en el mismo PR que reporta la corrida.** Un manifiesto que solo existe en disco local (el `.migration-manifest.<proyecto>.json` / `.recompute-manifest.<proyecto>.json` que `migrate.mjs`/`recompute.mjs` emiten por defecto) no es evidencia: depende de que quien corrió la herramienta lo mueva a mano bajo `evidencia-fase-4/` antes de que la sesión termine, y nada en el proceso ni en el código lo exige hoy.

**Caso fundante:** las dos únicas corridas reales que existen contra `dev-<proyecto>` no tenían su manifiesto en `origin/main` — una se había borrado en un commit posterior de la misma rama, la otra vivía sólo en una rama nunca mergeada. Los dos se recuperaron por arqueología de git (`git show <sha>:<path>` sobre commits todavía alcanzables), que es azar de retención de objetos, no un contrato (PR #162 (PR 162 del precursor), `fase-4-inventario-clon-legado.md` (precursor, `vc/teyst/docs/planning/fase-4-inventario-clon-legado.md` en `51fbd2a1`), hallazgo 4.1).

**Misma familia que "verificar contra una URL servida" arriba: un artefacto sin metadato de origen commiteado no es atribuible, sea una URL de deploy o una corrida de datos.** Si el commit que contiene el manifiesto se podara (`git gc` sin referencias, rama remota borrada), la corrida quedaría sin atribución verificable aunque haya ocurrido — la regla no depende de la suerte de la retención de objetos de git.

## El deploy sale de un checkout limpio de main

**Todo deploy corre desde un checkout limpio de `main`, nunca desde el worktree de una rama en progreso.** El worktree es donde se arregla; el checkout principal, sin cambios pendientes, es desde donde se despliega — así el sello que el build estampa (`teyst:commit`, arriba) coincide siempre con lo que `main` tiene en ese momento, y no con un estado intermedio de una rama que todavía no mergeó. Un deploy automatizado hace esto por construcción: dispara sobre `main`, corre en un runner efímero que nunca vio una rama, y aborta si el checkout no está limpio — ver `.github/workflows/teyst-deploy-dev.yml` para la instanciación.

_(Regla escrita el 2026-08-28, junto con la de `teyst:commit`: es la misma causa raíz — 39 releases de `hosting:dev`, todos manuales, sin metadato de origen. Sin un checkout de main verificado como precondición del deploy, el sello del build puede ser exacto y el deploy igual haber salido de un estado que no es main.)_

## CD con gates

El despliegue es **opt-in por proyecto**: se define acá y se activa cuando un proyecto lo pida. Ningún repositorio despliega por el solo hecho de adoptar el CI.

1. **Gate humano igual a `workflow_dispatch`.** Todo deploy se dispara a mano. En repos privados del plan Free no existen environments con revisores requeridos; el disparo manual es el gate, y para un portfolio de un integrador es suficiente.
2. **Credenciales solo por identidad federada.** OIDC con Workload Identity Federation hacia GCP. Ninguna clave de cuenta de servicio almacenada en los secrets del repositorio, según la [gestión de secretos](secrets.md).
3. **Dev antes que prod.** Prod exige un dispatch separado con un input explícito de confirmación. Un solo botón que despliega a producción es un accidente esperando su turno.
4. **Guardrail de billing vigente como precondición.** Ningún proyecto tiene su primer deploy automatizado sin budget guardrail activo ([invariante 37](role-matrix.md)): automatizar el despliegue es automatizar el gasto.

Instanciaciones previstas: Teyst desplegará Cloud Functions y reglas por esta vía —los builds de EAS se deciden en su propio ciclo—; Pinot Index publicará su biblioteca por tag, con release por `git tag` y CI verde, consumida con versión fijada.

## Compuerta de consola en el deploy automático

Cuando un proyecto agregue verificación automática de consola del navegador
como gate de su deploy (todavía no implementada, este documento fija la
regla antes de que exista código que la viole), valen dos condiciones.

**La compuerta corre siempre sobre una carga fresca, nunca sobre una sesión
con sign-out previo.** Una sesión que ya hizo sign-out arrastra estado de
browser storage que un visitante de verdad nunca tiene en su primera carga
— no es la condición que el deploy necesita validar, y mezclarla con la
carga fresca hace que un ruido de un caso no reproducible tape una señal
real del otro. La carga fresca es la línea de base reproducible; todo lo
demás es un chequeo distinto, con su propio gate si hace falta.

**Un error de tercero que no se puede arreglar no bloquea la compuerta —
pero solo entra por una lista de exclusión explícita, con el patrón exacto
y el motivo escrito al lado.** Sin motivo escrito, una exclusión es
indistinguible de una compuerta apagada: nada dice si alguien evaluó el
error y no lo puede resolver, o si lo silenció porque molestaba. El motivo
tiene que alcanzar para que otra persona (u otro agente) decida, sin
reabrir la investigación, si la exclusión sigue vigente.

_Caso de trabajo (2026-08-28): el login de Google en Teyst genera en
consola un warning de `Cross-Origin-Opener-Policy` que `accounts.google.com`
declara sobre sí mismo (`report-only`, medido con `curl -I`) — no sale de
`dev-<proyecto>.web.app` ni del `authDomain` de Firebase, y se descartó la
hipótesis de que el PR #94 (PR 94 del precursor)
lo hubiera introducido: su lista de cabeceras no declara COOP en ningún
lado. Detalle completo en `vc/teyst/docs/infrastructure-notes.md` §
Cross-Origin-Opener-Policy en el login. Es exactamente el caso que esta
regla contempla: en carga fresca sin sign-out previo la consola no muestra
ningún otro error, así que el warning de COOP es el único candidato a
exclusión._

**Patrón de exclusión, confirmado por captura del PO (2026-08-28), texto
literal de la consola:**

```
Cross-Origin-Opener-Policy policy would block the window.close call.
```

**El patrón que entra a la lista no es esa frase completa — es la
co-ocurrencia de `Cross-Origin-Opener-Policy` y `window.close`.** El texto
exacto es de Chrome y puede cambiar entre versiones; matchear por la frase
entera hace que la exclusión deje de aplicar el día que Chrome la
reformule sin que el warning haya dejado de ser el mismo. Motivo al lado,
como manda la regla de arriba: _"condición del proveedor — cabecera
declarada por `accounts.google.com` en su propio dominio, medida el
2026-08-28; revisado contra el PR #94 y contra `dev-<proyecto>.web.app`,
ninguno de los dos la declara; sin arreglo posible de nuestro lado."_

## Sin credenciales en verificación

**Ningún job de verificación usa credenciales ni consulta el estado de un servicio externo.** El único token que existe en el CI es el `GITHUB_TOKEN` que Actions emite por corrida, con los permisos mínimos que el workflow declara — fijados también a nivel del job `verify` (`contents: read`), para no depender de cómo un consumidor arbitrario resuelva la precedencia entre sus propios permisos y los del workflow reutilizable. La red sí se usa, y no es poco: el job baja la distribución de Node, el gestor de paquetes desde su registry, y el árbol completo de dependencias que el consumidor declare — cuyos scripts de instalación se ejecutan. Eso es código de terceros corriendo con los privilegios del job, no solo binarios de herramientas con checksum verificado. Lo que la regla realmente garantiza no es la ausencia de código ajeno — ese código corre, y el estándar lo asume — sino que no hay ninguna credencial en el entorno que ese código pueda robar.

**Los workflows de verificación se disparan con `pull_request`, nunca con `pull_request_target`.** La diferencia decide si un fork hostil ejecuta su código con los secretos del repositorio: `pull_request_target` corre en el contexto del repositorio base con acceso a ellos. La biblioteca evalúa `build-command` con `eval` por diseño, así que esa combinación convierte un pull request de un fork en ejecución remota de código.

**Las acciones de terceros se fijan por SHA, con la versión en un comentario.** Un tag de Git es movible: quien controla el repositorio de la acción puede reapuntarlo y cambiar lo que corre en el CI sin que cambie una línea del consumidor. Es la misma exigencia que ya rige para los binarios que el pipeline descarga y para el árbol de dependencias que instala, y por el mismo motivo: todo es código que corre con los privilegios del job. El costo aceptado es que actualizar deja de ser automático.

La regla se sostiene sola: **si un check necesita un secreto, no es un check — es un despliegue disfrazado de verificación**. Y separar ambas cosas es lo que permite que cualquier PR, venga de donde venga, ejecute la verificación completa sin exponer credenciales — no sin ejecutar código de terceros, que sí ejecuta, vía dependencias instaladas y `build-command`. Lo que ese código nunca encuentra es algo que valga la pena robar.

Consecuencias prácticas:

- Los validadores leen archivos versionados, no APIs. Lo que el CI verifica es el repositorio, no el estado del mundo.
- El job `verify` declara `permissions: contents: read` en su propia definición, no solo a nivel de workflow: un consumidor que declare permisos amplios al llamarlo no se los entrega de más a un job que hace `eval` de comandos declarados y corre scripts de ciclo de vida de dependencias de terceros.
- El workflow programado de renovaciones abre issues con el token de Actions y `issues: write`, sin credenciales nuevas ni servicios de correo.
- Un check que necesite consultar a un proveedor se ejecuta como job de despliegue, con su gate y su identidad federada, o no se ejecuta.

## Credenciales del builder

El builder abre PRs y empuja ramas, pero **no mergea**: el merge es acto del PO por SHA (o del gate del tren, solo con `BLENDS_TRAIN_AUTO_MERGE=1`). Hasta la decisión del 2026-10-13 esa regla la sostenían la lista de comandos prohibidos y la protección de `main`, mientras el builder corría con las credenciales del PO, que sí pueden escribir contenido. La decisión A del PO acota la credencial en lugar de confiar en la lista.

| Pieza         | Qué es                                                        | Alcance                                                                                                                                                                        |
| ------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Token de `gh` | Token fino (fine-grained) del PO, limitado a este repositorio | `Pull requests: Read and write`, `Contents: Read-only`, más lecturas que el builder usa (`Actions`, `Checks`, `Commit statuses` y `Variables`: Read-only). Vence a los 90 días |
| Deploy key    | Llave ssh del repositorio, con escritura                      | Empuja ramas; `main` lo frena la plataforma (checks requeridos estrictos y `enforce_admins`, verificado el 2026-10-13)                                                         |

**Lo crea el PO, no una sesión.** Un token real no se escribe en el repositorio ni en el chat.

1. **Token.** GitHub, Settings, Developer settings, Fine-grained personal access tokens: repositorio `denispaiva/blends` solamente, con los permisos de la tabla.
2. **Guardarlo en el Llavero**, sin dejarlo en el historial del shell (`security` pide el valor por la terminal):

   ```bash
   security add-generic-password -s blends-builder-gh -a "$USER" -w
   ```

3. **Deploy key.** Se crea sin contraseña, porque el builder corre sin terminal, y se carga en Settings, Deploy keys del repositorio con **Allow write access**:

   ```bash
   ssh-keygen -t ed25519 -N "" -C blends-builder -f ~/.ssh/blends-builder-deploy
   ```

4. **Verificar el token.** Tiene que dar las cuatro comprobaciones en verde (lee, abre PRs, no mergea, no escribe contenido):

   ```bash
   node vc/tools/verify-builder-credentials.mjs --from-keychain
   ```

5. **Verificar la llave:** `GIT_SSH_COMMAND="ssh -i ~/.ssh/blends-builder-deploy -o IdentitiesOnly=yes" git ls-remote git@github.com:denispaiva/blends.git` lista las referencias sin pedir nada.
6. **Cerrar la transición.** `export BLEND_RUN_REQUIRE_SCOPED=1` en el perfil del shell: desde ahí `blend-run` no lanza un builder sin las dos piezas. Hasta entonces avisa y sigue con las credenciales del PO.

Qué garantiza y qué no. El token no puede mergear ni escribir contenido aunque el builder quiera: la plataforma lo rechaza, que es la garantía que una lista de comandos prohibidos no da (`gh api -X PUT …/merge` quedaba fuera de ella). La deploy key sí puede empujar ramas, y por eso no se guarda donde el agente la lea: `Read(~/.ssh/**)` está negado en `.claude/settings.json`. El builder ve su propio entorno, así que el token acotado no es un secreto para él; lo que importa es lo que ese token puede hacer. Una GitHub App con identidad propia (opción B de la decisión) queda como evolución posible.

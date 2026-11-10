# Política de admisión de skills

Cómo entra una skill a la biblioteca del portfolio. Aplica a cualquier proyecto, no solo a este repositorio.

## Nombre

Al conjunto de reglas de este documento y al proceso que las aplica —evaluar un candidato de tooling, calcular su score y registrar la decisión humana— se lo llama **Scoring Skill**. No es una skill que se invoque dentro de una sesión: es el nombre del sistema completo, con dos piezas:

- **La política**: este documento (criterios de admisión, pipeline de estados, registro obligatorio) y `vc/tooling/scoring-policy.json` (precursor, `vc/tooling/scoring-policy.json` en `51fbd2a1`) (pesos por factor, umbrales de confianza, niveles de riesgo).
- **El proceso**: `vc/tools/evaluate-tooling.mjs` (precursor, `vc/tools/evaluate-tooling.mjs` en `51fbd2a1`) (CLI `evaluate`/`decide` que un agente corre paso a paso con evidencia real) y `vc/tools/validate-tooling.mjs` (precursor, `vc/tools/validate-tooling.mjs` en `51fbd2a1`) (el gate de CI que verifica que toda skill activa tenga su evaluación). El motor detrás de `evaluate`/`decide` vive en `vc/brix/packages/cli/scoring/engine.mjs` (precursor, `vc/brix/packages/cli/scoring/engine.mjs` en `51fbd2a1`) desde el registro de decisión 0003 (precursor, `vc/brix/docs/decisions/0003-scoring-motor.md` en `51fbd2a1`) — `evaluate-tooling.mjs` es un envoltorio delgado sobre él; `validate-tooling.mjs` y este documento no se movieron.
- **El punto de entrada**: el comando `/tool <url>` (precursor, `.claude/commands/tool.md` en `51fbd2a1`), que es el guion con el que una sesión aplica todo lo anterior a un candidato concreto — qué evidencia traer y con qué rigor, cómo correr el CLI, dónde está el corte en el que decide el humano.

Cada decisión queda registrada en `vc/tooling/evaluations/` (precursor, `vc/tooling/evaluations` en `51fbd2a1`) (un `.evaluation.json` por candidato, con su historial) y resumida en `docs/tooling/evaluations.md` (precursor, `vc/docs/tooling/evaluations.md` en `51fbd2a1`).

Las fuentes de las que salen las candidatas —catálogos, listas, repositorios— se registran en `docs/tooling/candidate-sources.md` (precursor, `vc/docs/tooling/candidate-sources.md` en `51fbd2a1`); una fuente no se puntúa, se la consulta.

## Por qué existe esta política

Una skill no es una biblioteca que se invoca cuando conviene: son **instrucciones que entran al contexto del agente y condicionan cómo trabaja**, con acceso a las mismas herramientas que el agente. Admitir una skill es delegar autoridad, no agregar una dependencia. Por eso se evalúa antes de instalar, se instala acotada y se registra con evidencia, igual que cualquier otra decisión con consecuencias.

El contrato de seguimiento ya lo previó: la definición de `skill` acepta `installRef`, `evidence`, `owner` y `scope`. Esta política los pone en uso.

## Criterios de admisión

Los cinco primeros son marcadores reconocidos de una skill bien construida; los dos últimos responden al riesgo específico de delegar autoridad.

1. **Activación específica.** La descripción dice cuándo usarla, no qué es. Un disparador vago se activa donde no corresponde o no se activa nunca.
2. **Lo determinista, en código.** Parseo, validación y ordenamiento resueltos por scripts, no por criterio del modelo.
3. **`SKILL.md` breve.** Instrucciones centrales cortas; los casos borde viven en archivos que se cargan bajo demanda.
4. **Un solo trabajo.** Las skills compuestas se disparan mal.
5. **Ejemplos antes que reglas.** Tres ejemplos resueltos valen más que veinte restricciones enumeradas.
6. **Procedencia declarada.** Autor, repositorio y versión fijada. Las fuentes oficiales se prefieren, pero no se dan por buenas: también se leen.
7. **Sin llamadas de red no documentadas.** Se verifica leyendo la skill, no confiando en su descripción.

Señales de alarma que bastan para rechazar: archivos de miles de tokens que inflan el contexto sin necesidad, disparadores genéricos del tipo "para tareas de productividad", afirmaciones sobre código que no se pueden comprobar, y ausencia de ejemplos.

## Qué no se puntúa

No todo el tooling declarado pasa por el scoring. Hay una clase de item para la
que **no hay decisión que tomar**: no se lo eligió entre alternativas, es la
plataforma sobre la que se trabaja. Puntuar `git` no produce información — no
hay un mundo donde el score salga bajo y se cambie de herramienta.

Esa clase se declara con el campo `tier` en `tooling.technologies`, con dos
valores según de qué eje se hable:

| `tier` | Qué es | `version` | `owner` | `assessmentRef` |
| --- | --- | --- | --- | --- |
| `substrate` | Binario instalado que es sustrato del trabajo, del framework o de un proyecto concreto | fijada | obligatorio | exento |
| `platform` | Servicio SaaS sobre el que se trabaja: `github`, `github-actions`, `bigquery`, `google-sheets`, `apps-script` | `"no-aplica"` | obligatorio | exento |

`substrate` tiene dos escalas, con el mismo argumento en las dos:

- **Del framework**: lo que sostiene el trabajo de cualquier proyecto del
  portfolio — `node`, `git`, `gh-cli`, `gcloud`, `python`, `clasp`. Se declara
  en `vc/framework/.roadmap.json`.
- **De un proyecto**: el lenguaje, el runtime, el framework base y el SDK de
  la plataforma que ese proyecto eligió — lo que no se puede reemplazar sin
  reescribir el proyecto o rearmar su estructura. En Teyst: `teyst-typescript`
  (el lenguaje), `cloud-functions-node-runtime` (el runtime),
  `teyst-firebase-functions`/`teyst-firebase-admin`/`teyst-firebase-js-sdk`
  (el framework y los SDK de la plataforma elegida, Firebase). Decisión del
  PO, 2026-09-14.

El corte, en ambas escalas, es si es reemplazable sin tocar el producto. Lo
que sigue siendo una elección entre alternativas —el test runner
(`teyst-vitest`), el pipeline de build (`eas-build-submit`), una integración
(`whatsapp-cloud-api`), un control de seguridad opcional
(`firebase-app-check`)— se evalúa igual que cualquier otro candidato, aunque
viva en el mismo `category` o dependa de la misma plataforma que un ítem
sustrato. `gcloud`, `gh-cli` y `clasp` son `substrate` aunque cada uno sea el
CLI de un SaaS: `gcloud 554.0.0` es un binario concreto en disco, no el
servicio. Por el mismo argumento, `firebase-cli` —el CLI, no el servicio— es
`substrate` en `vc/framework/.roadmap.json` desde 2026-09-14: nadie va a
elegir otro CLI de Firebase.

Un servicio SaaS no tiene versión que fijar, pero **declara el campo igual**,
con el centinela `"no-aplica"`. Omitirlo no distingue "no aplica" de "se lo
olvidaron" — el mismo principio con el que el archivo de evidencia exige
`availability: "unavailable"` explícito en vez de un subfactor ausente. El
validador rechaza las dos incoherencias: un `platform` sin centinela y un
`platform` con una versión real (que sería `substrate`).

**La exención es del scoring, no del registro.** Los dos tiers siguen
declarando `owner`, y `substrate` sigue declarando versión fijada. Es
deliberado: la decisión de runtime de Node es gobernanza de versión sobre un
item que nadie va a puntuar nunca, y sin esos campos no cuelga de ningún lado.

**Sólo vale en `technologies`.** Un `tier` en `tooling.skills` o
`tooling.mcps` es error de validación, por schema y por validador. Una skill es
instrucción que condiciona cómo trabaja el agente y un MCP es un conector de
terceros con credenciales: son exactamente lo que esta categoría no puede
eximir.

**El criterio se argumenta, no se infiere.** No hay umbral de popularidad ni de
antigüedad que admita a un item por su cuenta. `category` —la taxonomía técnica
del inventario— no sirve como atajo, porque cruza el límite: `tooling`
contiene a `git` (sustrato) y a `graphify` (evaluada), y `testing` contiene a
`teyst-firebase-functions-test` (sustrato — es la utilidad oficial para
testear las Cloud Functions de Teyst, no hay otra) y a `teyst-vitest`
(evaluada — el test runner sí es una elección entre alternativas). Que algo
sea público y consolidado no lo exime: `gitleaks` es de terceros y hay
alternativas reales, así que se evalúa.

## Pipeline

| Estado | Alcance | Significa | Para avanzar |
| --- | --- | --- | --- |
| `planned` | — | Evaluada y aprobada (`decide` la registra con `registeredAt`), sin evidencia suficiente de uso | Uso medido: ≥3 sesiones en ≥2 días dentro de los últimos 30, posteriores al registro (`validate-tooling` lo informa; decide el PO) |
| `planned` | `task` | **Cuarentena**: instalada y usada en un trabajo real de bajo riesgo | Lo mismo, con el checklist de los siete criterios aprobado |
| `active` | `project` | Admitida para un proyecto | Uso sostenido sin fricción |
| `active` | `workspace` | Promovida a todo el portfolio | — |
| `deprecated` → `removed` | — | Baja, con motivo registrado | — |

**Ninguna skill se instala primero a nivel usuario.** Entra con el alcance más chico que permita evaluarla y se promueve después. Es la misma lógica del guardrail de presupuesto: exposición limitada antes que confianza.

**Una skill horizontal vive una vez, en framework.** Un proyecto que la usa
no la re-declara: la referencia en su `tooling.skills` con
`{ "ref": "framework#<id>", "purpose": "<para qué la usa este proyecto>" }`
(`schemaVersion` 1.13.0). Estado, versión y evaluación quedan en la entrada
de framework; `validate-tooling` falla si una evaluación aprobada no
resuelve a exactamente una entrada canónica o si una referencia no resuelve.
Si el tipo evaluado no coincide con el arreglo donde vive la entrada (V4),
solo advierte: hoy `impeccable` y `markitdown`. Las evaluaciones `practice`
están exentas del invariante. Límite documentado: `validate-governance` no
avisa si una tarea abierta referencia una herramienta de framework
`deprecated` (su V2 solo cubre `removed`).

## Registro obligatorio

Toda skill en estado `active` declara en el inventario `tooling.skills` de su `.roadmap.json`:

- **`installRef`**: la referencia fijada que se evaluó, con versión o fecha (`claude-plugins-official/superpowers@6.2.0`). Sin versión no hay evaluación reproducible.
- **`evidence`**: enlace a su sección en `docs/tooling/evaluations.md`, con el checklist resuelto y el veredicto.
- **`owner`**: quién responde cuando la skill se actualiza, falla o hay que darla de baja.

`node tools/validate-skills.mjs` verifica estas tres condiciones y falla si alguna falta.

## Revisión

Cada evaluación registra su fecha. El validador advierte cuando una skill activa lleva más de 90 días sin revisar, porque `npx skills update` cambia el contenido sin cambiar el nombre: lo que se evaluó y lo que corre pueden dejar de ser lo mismo.

Una actualización que cambia la versión fijada exige repetir el checklist. Una skill que ya no se usa se marca `deprecated` con motivo, no se borra en silencio.

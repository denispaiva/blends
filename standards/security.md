# Seguridad de aplicación — marco operativo

> **Estado: decidido, no vigente.** Las once decisiones del PO sobre este
> documento están tomadas, con fecha (D1-D11, 2026-08-26 —
> `security-adoption.md` (precursor, `vc/docs/standards/security-adoption.md` en `51fbd2a1`), sección 7). Lo que falta es
> la **migración formal**: mover las invariantes 45 y 46 a
> [`role-matrix.md`](role-matrix.md), cambiar este mismo encabezado a
> "adoptado", y declarar las tareas y criterios de cierre en los
> `.roadmap.json` correspondientes. Esa migración está **diferida, no
> ausente**: trackeada en
> PR #98 (PR 98 del precursor). Hasta que cierre,
> **ninguna regla, invariante ni gate de este documento rige todavía** — están
> decididos, no vigentes. Es la misma distinción que la sección "Los dos
> estados de un control no implementado" propone más abajo, aplicada a la
> adopción de este documento consigo mismo.

Cómo se protegen los **datos y las superficies expuestas** de cualquier proyecto
del portfolio. Es el complemento faltante de la seguridad de cadena de
suministro, que el estándar de gobernanza ya cubre en las
[invariantes 38, 39 y 42](role-matrix.md) y en
[la gestión de secretos](secrets.md).

## Por qué existe este documento

El portfolio codifica con detalle cómo se protege el **repositorio** —secretos,
publicación, integración— y casi nada de cómo se protege la **aplicación**. Esa
asimetría no es una opinión: hasta el 2026-08-26 no existía este archivo.

Mientras tanto, Teyst resolvió bien varios de esos controles. El problema es
**dónde** los resolvió: en su código y en un documento de assessment propio. Un
proyecto nuevo en `vc/` hereda los hooks de gitleaks, el pipeline y nueve
validadores de gobernanza, y **no hereda una sola línea** de deny por defecto,
allowlist de campos o censo de superficie. Lo que hoy protege a Teyst es
conocimiento de Teyst.

Este documento propone mover ese conocimiento al marco, con la misma anatomía
que [`secrets.md`](secrets.md): principio, reglas concretas, y el mecanismo
que las verifica.

## Principio

**La autorización se declara donde el dato vive, y se verifica donde el dato se
toca.** Un permiso que solo existe en el cliente es una sugerencia; un permiso
que solo existe en una revisión es una intención.

De ahí se desprende el reparto de este estándar: lo que se puede leer de un
archivo versionado —una regla de datos, un decorador de función, una cabecera
declarada— se verifica en una máquina. Lo que exige entender qué hace el código
—si esa validación cubre el caso, si ese motivo es razonable— queda para la
revisión, que es donde debe estar.

Y una regla de reparto heredada de [`ci-cd.md`](ci-cd.md): **si un check se
puede escribir, se escribe**. Agregar un párrafo a este estándar sin agregar su
verificación es documentar un deseo — que es exactamente lo que le pasó al
encabezado "Seguridad y privacidad" de
`architecture.md` (precursor, `vc/docs/templates/architecture.md` en `51fbd2a1`), vivo desde hace meses con sus
cuatro viñetas vacías.

## Alcance: qué proyecto queda alcanzado por qué

Ninguna regla de este estándar aplica a todos los proyectos por igual, y fingir
que sí produce el peor resultado posible: un proyecto que declara cumplimiento
de una regla que no le aplica, y otro que la necesita y no la ve.

Cada proyecto declara su **superficie** en su archivo de arquitectura, con una
de tres respuestas por cada superficie posible:

| Superficie | Qué la define | Reglas que activa |
| --- | --- | --- |
| **Datos** | Existe un motor de datos con reglas de acceso del lado del servicio (Firestore, Storage, RLS) | R1, R2, R4, R7 |
| **Funciones expuestas** | Existe código alcanzable por la red desde fuera del proyecto | R3, R4, R5 |
| **Web** | Existe HTML servido a un navegador | R6 |

Una superficie que **no existe** se declara `no aplica` **con el motivo escrito
y el disparador que la vuelve a poner sobre la mesa**. Un `no aplica` sin motivo
se convierte en un `se nos pasó` en cuanto la arquitectura cambia, y nadie se
entera porque el documento sigue diciendo lo mismo.

## Las siete reglas

Cada regla declara tres cosas: **qué exige**, **cómo se verifica** —mecánico,
por revisión, o sin gate a propósito— y **si sube a invariante numerada o se
queda como convención de este estándar**.

### R1 — Deny por defecto en reglas de datos

Todo motor de datos con reglas del lado del servicio termina con una regla
terminal que niega todo lo no concedido explícitamente antes.

```
match /{document=**} {
  allow read, write: if false;
}
```

**Por qué:** sin la regla terminal, una colección nueva nace accesible. El
descuido no se ve en ninguna revisión, porque el archivo no cambió: lo que
cambió es que apareció un dato que nadie nombró.

**Gate: mecánico.** `vc/tools/check-security-rules.mjs`, en el job `governance`.
Falla si un proyecto con superficie de datos declarada no tiene archivo de
reglas, o si el archivo no termina con la regla terminal de negación.

**Invariante: sí — propuesta como invariante 45.** Es la base sobre la que
descansan todas las demás reglas de datos, es independiente de la tecnología
—Firestore, Storage y RLS de Postgres expresan lo mismo con otra sintaxis— y su
violación no tiene grados: o el default niega o no niega.

### R2 — Allowlist de campos en escritura, nunca denylist

Toda escritura permitida enumera los campos que acepta. Nunca los que rechaza.

```
request.resource.data.keys().hasOnly(['id', 'provider', 'created_at'])
request.resource.data.diff(resource.data).affectedKeys().hasOnly(['name', 'updated_at'])
```

**Por qué:** una denylist protege contra los campos que quien la escribió
imaginó. El campo que se agrega el mes que viene entra solo, y entra con los
permisos de quien escribe, no con los de quien diseñó la regla.

**Gate: mecánico, en modo advertencia y con un falso positivo ya identificado.**
El check exige que toda concesión de escritura cuya condición no sea `false`
mencione `hasOnly(`. La regla `isValidUserUpdate` de Teyst **no** lo menciona y
sin embargo es más estricta que cualquier allowlist —exige
`request.resource.data == resource.data`, es decir inmutabilidad total—. Ese
caso es un falso positivo real, conocido antes de escribir el check, y es la
razón por la que nace en advertencia.

**Invariante: no, como cláusula de la 45.** Es la misma regla aplicada a un
grano más fino: el default niega, también a nivel de campo. Sigue el precedente
de la precisión de la [invariante 44](role-matrix.md), que vive como cláusula
y no como número nuevo porque no es otra regla.

### R3 — Validación de entrada antes del primer acceso a datos

Toda función expuesta valida su entrada —forma, tipo, rango— **antes** de la
primera lectura o escritura de datos.

El ejemplar vive en el repositorio: `resolveEventCode` canonicaliza el código y
lanza `invalid-argument` antes de tocar Firestore.

**Por qué:** un dato sin validar que llega a una consulta es la superficie de
inyección clásica; y una función que consulta antes de validar convierte
cualquier basura en una lectura facturable. El orden importa tanto como la
validación.

**Gate: de revisión.** Fila nueva en
`code-review.md` (precursor, `vc/docs/templates/code-review.md` en `51fbd2a1`), que hoy tiene una fila
"Seguridad" sin un solo criterio debajo. No hay gate mecánico y **no se propone
uno improvisado**: verificar "antes de" exige seguir el flujo de datos, no
buscar un texto. Un check de texto que aproximara esto daría falsos negativos
silenciosos, que es la peor clase de check.

**Herramienta candidata:** un SAST con reglas de flujo (Semgrep o equivalente).
No se instala acá: pasa por la [política de admisión](skills.md) y por
`vc/tools/evaluate-tooling.mjs`, como cualquier herramienta nueva
([invariante 40](role-matrix.md)).

**Invariante: sí — invariante 46** (decisión del PO, D3: R3 y R5 quedan
**separadas**, no juntas en un mismo número — ver el motivo en R5). Que no
tenga gate mecánico no la descalifica: las invariantes 43 y 44 tampoco lo
tienen, y son de las más citadas del estándar. Una invariante declara qué es
innegociable; el gate es una pregunta separada, y este documento la responde
por separado.

### R4 — Toda apertura sin sesión, declarada con motivo en el archivo

Una regla de datos que concede acceso anónimo, y una función expuesta que no
exige sesión, llevan su motivo **en el mismo archivo donde viven**.

**Por qué el mismo archivo y no un documento aparte:** un documento de
seguridad separado envejece sin que nada lo diga. El comentario al lado de la
línea se lee justo cuando alguien está por copiarla, que es el momento en que el
motivo importa. Y cuando la línea se borra, el motivo se borra con ella — un
documento aparte se queda describiendo una apertura que ya no existe.

**Gate: mecánico, en modo advertencia.** El check exige un comentario en la
línea inmediatamente anterior a toda concesión anónima, y en la definición de
toda función expuesta sin verificación de sesión.

**Estado real hoy, verificado:** `vc/teyst/firestore.rules` tiene **ocho
concesiones anónimas en siete rutas** —líneas 142, 157, 160, 167, 172, 197, 202
y 203— y **ninguna** tiene comentario de motivo. Del lado de las funciones, en
cambio, la regla ya se cumple: las cuatro excepciones públicas están
documentadas en código. Nacer bloqueante rompería `main` en el acto.

**Invariante: no, convención de este estándar.** Es una regla de trazabilidad,
no de autorización: una apertura sin comentario es igual de abierta que una
comentada. Su valor es que la próxima persona entienda; eso es exactamente lo
que este estándar existe para fijar, y no necesita un número para hacerlo.

### R5 — Censo de superficie expuesta como criterio de cierre de fase

Ninguna fase de un proyecto con funciones expuestas cierra sin repetir el censo
de autorización: función por función, qué exige cada una, contra el corte
anterior.

**Por qué como criterio de cierre y no como tarea:** una tarea se hace una vez.
El censo que hoy vive en
`security-finding-public-callables-review.md` (precursor, `ideas/teyst/assessment/security-finding-public-callables-review.md` en `51fbd2a1`)
es el mejor material de seguridad del repositorio —23 callables auditadas una
por una, 21 coincidencias byte a byte, 2 divergencias reportadas aunque ninguna
empeorara la seguridad— y existe porque alguien lo pidió expresamente. Se hizo
por decisión, no por proceso. La próxima fase no lo repite salvo que alguien se
acuerde.

El censo produce, además, la distinción que la sección siguiente formaliza:
separa lo diferido con motivo de lo que nadie miró nunca.

**Gate: de revisión, anclado en el cierre de fase.** El criterio de salida de
toda fase que toque funciones expuestas incluye el censo. Un `.roadmap.json` que
declare fase cerrada sin censo es lo que un check futuro podría detectar; hoy no
existe y este documento no finge que sí.

**Invariante: no** (decisión del PO, D3). R3 es una regla de código, verificable
función por función; R5 es una regla de proceso anclada al cierre de fase.
Juntarlas en una sola invariante 46 haría que citarla quedara ambiguo sobre
cuál mitad se invoca — que "nadie citaría las dos por separado" era una
afirmación de este documento, no evidencia, y el costo de equivocarse es una
invariante que se cita mal. R5 **queda como criterio de cierre de fase**, que
para Teyst ya lo es por decisión previa del PO (ver "Estado real hoy" arriba:
el censo de `security-finding-public-callables-review.md` existe porque el PO
lo pidió como cierre de Fase 3). Este estándar generaliza esa decisión ya
tomada a toda fase que toque funciones expuestas — no inventa un criterio
nuevo, documenta uno que ya rige para Teyst.

### R6 — Cabeceras de seguridad declaradas por proyecto con superficie web

Todo proyecto que sirva HTML declara sus cabeceras de seguridad en el archivo de
configuración de su hosting, versionado.

Línea base a **declarar en el archivo**: `Content-Security-Policy`,
`X-Content-Type-Options: nosniff`, `Referrer-Policy` y `Permissions-Policy`.

**`Strict-Transport-Security` queda fuera de la lista de declaración
(corrección del PO, verificada contra el sitio real, 2026-08-26).** No es que
no importe: es que la regla tiene que enunciarse sobre lo que el **navegador
recibe**, no sobre lo que el archivo declara, y Firebase Hosting la manda por
default. Medido en `dev-<proyecto>.web.app`: `strict-transport-security:
max-age=31556926; includeSubDomains; preload` presente en la respuesta sin
una sola línea de HSTS en ningún `firebase.json` del repositorio. Exigir
declararla abre un riesgo real y no teórico: alguien escribe un `max-age` más
corto que el default de la plataforma y la **debilita** creyendo que la
cumple. **Disparador que la reabre:** cambiar de proveedor de hosting — ahí
deja de haber un default garantizado y la cabecera vuelve a la lista de
declaración.

**Por qué las que quedan:** son la única defensa que actúa sobre un navegador
ya comprometido por contenido de terceros, y no cuestan nada más que
declararlas.

**Gate: mecánico, en modo advertencia.** El check verifica que la configuración
de hosting declare el bloque de cabeceras y que la lista base esté completa. No
verifica que la política sea **buena** —una CSP con `unsafe-inline` pasa igual—
y eso queda dicho acá para que nadie lea el check verde como una CSP revisada.
**El check tiene que poder distinguir "el archivo no la declara" de "la
plataforma ya la manda"** — si no, produce falsos negativos en la peor forma
posible: exige declarar (o corregir) una cabecera que ya está bien, y quien
responda a esa presión puede terminar escribiendo un valor peor que el
default que reemplaza. Esto es una condición de diseño para cuando se
construya `check-security-rules.mjs` (D8: después), no algo que el check ya
resuelve — no existe todavía.

**Criterio de revisión acompañante (D9, agregado tras un caso real, precisado
por el PO):** el check mecánico de arriba detecta la **presencia** de los
nombres de cabecera, no su efecto — y ese hueco no es teórico. El disparador
de la prueba manual es: **`X-Frame-Options`, con cualquier valor** (se aplica
siempre, sea o no la CSP `Report-Only`) **o una CSP en modo enforce** —
`Content-Security-Policy`, no `-Report-Only`— **que incluya `frame-ancestors`
o `script-src`**. Precisión necesaria: una CSP en `Report-Only` con
`frame-ancestors` **no dispara nada por sí sola** — en ese modo el navegador
reporta la violación pero no bloquea, así que no hay riesgo de framing real
por ese lado mientras la política siga en `Report-Only`.

Exige una prueba real del flujo de autenticación en un navegador antes de
mergear — no alcanza con `curl -I`. Motivo: las cabeceras se verifican con una
petición HTTP; el login de Firebase con `signInWithPopup` se rompe en el
navegador, dentro de un popup o un iframe que ninguna petición HTTP ejercita —
un `X-Frame-Options: DENY` a nivel `**` que le quede aplicado a `/__/**` (donde
viven `/__/auth/handler` y `/__/auth/iframe`) pasaría el check de R6 en verde y
rompería el login igual — y esto no depende del modo de la CSP, porque
`X-Frame-Options` es una cabecera separada. Un entorno automatizado tampoco
sirve para esta prueba: `window.open()` devuelve `null` en un navegador
sandboxeado y el popup nunca llega a abrirse. La prueba la hace una persona,
en un navegador real. Fila correspondiente en
`code-review.md` (precursor, `vc/docs/templates/code-review.md` en `51fbd2a1`), sección 5.

**Estado real hoy, verificado (2026-08-26):** `vc/teyst/firebase.json` declara
`hosting` con cabeceras de `Cache-Control` (PR #95, ya mergeado) pero
**ninguna** de la línea base de R6 a declarar en archivo —
`Content-Security-Policy`, `X-Content-Type-Options`, `Referrer-Policy` ni
`Permissions-Policy`. (`Strict-Transport-Security` no cuenta como falta: la
sirve la plataforma, ver arriba.) `Cache-Control` es una familia de cabecera
distinta; mergear #95 no mueve esta fila. El PR que sí declara la línea base
es el #94 (PR 94 del precursor), abierto, sin
mergear — y, verificado contra su diff, su CSP nace en modo
`Content-Security-Policy-Report-Only`, no en enforcement, **sin `report-uri`
ni `report-to` en ninguna de sus tres variantes** — sin endpoint, ese
`Report-Only` no reporta a ningún lado, y sin evidencia de violaciones reales
no hay sobre qué base promoverlo a enforce. Hueco de #94, no de este
documento ni de este PR.

**Invariante: no, convención de este estándar.** Solo aplica a proyectos con
superficie web. Una invariante numerada se lee como universal, y una universal
que en la práctica exceptúa a la mitad del portfolio enseña a leer las
invariantes como sugerencias.

### R7 — Restricción de archivos por dueño, tamaño y tipo

Toda escritura de archivos acota las tres dimensiones: quién escribe, cuánto
pesa y de qué tipo es.

El ejemplar también está en el repositorio, en `vc/teyst/storage.rules`: dueño
por `request.auth.uid == uid`, tamaño por `< 10 * 1024 * 1024`, tipo por
`contentType.matches('image/.*')`.

**Por qué las tres:** sin dueño, cualquiera escribe en el espacio de cualquiera.
Sin tamaño, la factura es la vulnerabilidad. Sin tipo, el bucket es un servidor
de contenido arbitrario con el dominio del proyecto delante.

**Gate: mecánico, en modo advertencia.** El check exige las tres condiciones en
toda concesión de escritura de archivos. **Teyst ya lo pasa**; nace en
advertencia por acompañar al resto del validador, no porque falle.

**Invariante: no, convención de este estándar.** Mismo argumento que R6: aplica
solo a proyectos con almacenamiento de archivos.

## Los dos estados de un control no implementado

Un control que no está puede estar en dos situaciones muy distintas, y el
portfolio hoy **no las distingue**. Distinguirlas es la mitad del valor de este
estándar.

| Estado | Qué significa | Qué exige |
| --- | --- | --- |
| **Diferido** | Alguien lo miró, decidió no hacerlo ahora y escribió por qué | Motivo, disparador concreto que lo reabre, y ubicación del registro |
| **Abierto** | Nadie lo miró | Dueño y fecha de revisión. No se queda así |

Lo que **no** es un estado válido: la ausencia. Un control que no aparece en
ningún documento no está diferido — está invisible, y la diferencia se nota
recién cuando alguien pregunta y nadie sabe.

El caso real que da origen a esta sección, verificado en el repositorio:

- **Diferidos, bien registrados:** App Check en las callables no intencionalmente
  públicas, y los cinco controles operacionales —rate limiting, alertas,
  auditoría IAM, pruebas de abuso, procedimiento de revocación—. Los seis tienen
  motivo escrito, evidencia que lo sostiene y secuenciación explícita en el
  censo de Teyst.
- **Abiertos sin traza:** cabeceras de seguridad y límite de intentos de login.
  Verificado: `firebase.json` no declara cabeceras, y `vc/teyst/` no contiene
  ninguna implementación de límite de intentos. Ninguno de los dos aparece en
  ningún documento del proyecto.

Los primeros seis son una decisión. Los últimos dos son un hueco. Hoy se ven
igual.

## Modo de nacimiento de cada gate

El repositorio ya pagó las dos lecciones que fijan esta sección: el job `lint`
nació bloqueante sobre 122 archivos sin formatear, y `check-branch-scope` nació
en `continue-on-error` con criterio de promoción explícito y un
log de observaciones (precursor, `vc/docs/tooling/branch-scope-observations.md` en `51fbd2a1`) que lo sostiene.

**La regla que se propone: un gate nace bloqueante solo si el árbol actual ya lo
pasa.** No es una preferencia de estilo — es la diferencia entre un gate y una
rama rota. Y se verifica antes de escribirlo, no después.

| Check | Modo al nacer | Por qué | Qué lo vuelve bloqueante |
| --- | --- | --- | --- |
| Regla terminal de negación (R1) | **Bloqueante** | Teyst la tiene en sus dos archivos de reglas. Verificado | Nace así |
| Allowlist en escritura (R2) | Advertencia | Falso positivo conocido: `isValidUserUpdate` | Que el check distinga inmutabilidad total de ausencia de allowlist, más el criterio de PRs limpios que fije el PO |
| Motivo en apertura anónima (R4) | Advertencia | Ocho concesiones sin comentario en Teyst | Las ocho documentadas |
| Cabeceras web (R6) | Advertencia | Teyst no declara ninguna | Cabeceras declaradas en `firebase.json` |
| Restricción de archivos (R7) | Advertencia | Acompaña al validador | Que el validador entero se vuelva bloqueante |

**Quién evalúa la promoción: el PO**, con la evidencia del log de observaciones
del validador, siguiendo el mismo mecanismo del período de advertencia de
`check-branch-scope`.

## Qué garantiza el validador propuesto, y qué no

Exigencia de la [invariante 43](role-matrix.md): una afirmación sobre lo que
un guard detecta se escribe con el alcance que la búsqueda de contraejemplos
sostiene.

**No hubo búsqueda de contraejemplos: el validador todavía no existe.** Lo que
sigue son las mutaciones que ya se ven a simple vista y que lo **atravesarían**,
escritas antes de construirlo para que el alcance quede fijado desde el
principio y no se ensanche solo:

- **R1.** El check detecta la **ausencia** de la regla terminal, no la
  efectividad del deny por defecto. Un `match /{a}/{b=**}` con `allow read: if
  true` colocado antes de la terminal pasa el check y abre el árbol entero.
- **R2.** Detecta la **ausencia del literal** `hasOnly(`. Un
  `hasOnly(request.resource.data.keys())` —tautología que no restringe nada—
  pasa. Que exista una allowlist no dice que sea correcta.
- **R4.** Detecta la **ausencia de un comentario**. Un comentario que diga
  `// abierto` pasa igual que uno que explique el motivo. La calidad del motivo
  es de la revisión, y así queda dicho en la fila de `code-review.md`.
- **R6.** Detecta la **presencia de los nombres** de cabecera. Una CSP con
  `unsafe-inline` o un `max-age=0` en HSTS pasan.
- **R7.** Detecta las **tres condiciones**. Un límite de tamaño de 10 GB las
  satisface.

Ninguna de estas limitaciones justifica no escribir el check: detectar la
ausencia total de un control es exactamente lo que hoy no detecta nadie. Pero
**la afirmación correcta es "detecta ausencia", no "detecta debilidad"**, y el
mensaje de error de cada check debe decirlo en esos términos.

## La capa que no puede verificar nada

Regla heredada de [`ci-cd.md`](ci-cd.md), que el pipeline aprendió tres veces
por las malas: **una capa de verificación que no puede verificar nada falla, no
pasa.**

El validador propuesto la instancia así:

- Un proyecto que declara superficie de datos y no tiene ningún archivo de
  reglas: **falla**, nombrando el proyecto.
- Un proyecto que declara superficie web y no tiene configuración de hosting:
  **falla**.
- Cero proyectos alcanzados por el validador: **falla**. Un portfolio donde
  ningún proyecto declara superficie es un error de declaración, no un portfolio
  seguro.

## Qué no cubre este estándar

Dicho explícitamente, porque un estándar de seguridad que no declara sus bordes
se lee como si los cubriera:

- **Seguridad de cadena de suministro.** Ya vive en las
  [invariantes 38, 39 y 42](role-matrix.md) y en [`secrets.md`](secrets.md).
- **Controles operacionales en tiempo de ejecución** —rate limiting, alertas,
  procedimiento de revocación—. Dependen de una superficie desplegada; su lugar
  natural es el gate de despliegue de [`ci-cd.md`](ci-cd.md), no un check
  sobre archivos versionados.
- **Privacidad y tratamiento de datos personales.** Es un eje distinto, con otra
  autoridad y otro marco regulatorio.
- **La corrección de cualquier política concreta.** Este estándar verifica que
  las decisiones estén declaradas y sean visibles. Que una CSP sea buena, que un
  motivo sea razonable o que una allowlist cubra el caso lo decide una persona
  leyendo.

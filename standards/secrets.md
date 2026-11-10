# Gestión de secretos — marco operativo

Cómo se aplica en la práctica la jerarquía de secretos y el guardia de secretos —previo al commit y previo al push— definidos en el [estándar de gobernanza](role-matrix.md). Vale para cualquier proyecto del portfolio, no solo para este repositorio.

## Principio

Preferir **identidad efímera** a **clave almacenada**. Un secreto que no existe no se filtra, no se rota y no se audita. La expiración periódica de un token es señal de que el mecanismo funciona.

## Jerarquía por caso de uso

| Caso | Mecanismo | Ejemplo vigente |
| --- | --- | --- |
| Operación local contra una API de Google | Impersonación de cuenta de servicio | `finops-sync@blends-ops` sin archivo de clave: `gcloud auth print-access-token --impersonate-service-account=... --scopes=...` |
| CI hacia GCP | Workload Identity Federation con OIDC | Pendiente al configurar el primer pipeline |
| Runtime de aplicación | Secret Manager | Pendiente hasta el primer proyecto desplegado |
| Automatización en Apps Script | Script Properties | `GCP_BIGQUERY_PROJECT`, `GCP_BILLING_TABLE`, `ALERT_EMAIL` |
| Desarrollo local | `.env` ignorado + `.env.example` versionado | Convención declarada en el README de `vc` |
| Secreto que teclea una persona | Gestor de contraseñas | Fuera del repositorio y de los chats |
| CI que consulta la API de GitHub entre repos | fine-grained PAT como secret de Actions | `GH_SCARCITY_PAT` (monitor de escasez) |

Cuando ninguna alternativa evita descargar una clave de cuenta de servicio, la excepción se documenta con ubicación fuera del repositorio, política de rotación y fecha de baja.

**Excepción vigente — `GH_SCARCITY_PAT`.** No hay identidad efímera disponible: el `GITHUB_TOKEN` nativo de un workflow solo ve el repositorio que lo emite, y el monitor de escasez (`github-scarcity.md` (precursor, `vc/docs/standards/github-scarcity.md` en `51fbd2a1`)) necesita leer los Actions de toda la cuenta `denispaiva` para medir el cupo compartido. El token es fine-grained, alcanza todos los repos de la cuenta y declara el mínimo necesario: Metadata read, Actions read, Variables read/write. Rotación por expiración, nunca por extensión — se regenera y se reemplaza con `gh secret set GH_SCARCITY_PAT -R denispaiva/blends`; el deadline `gh-scarcity-pat-expira` en `vc/tooling/lifecycle/deadlines.json` avisa antes del vencimiento. Baja: si el monitor se retira, se borran el secret y el token.

## Compuerta y sombra

El escaneo de secretos tiene dos herramientas con papeles distintos, y el papel lo decide un solo interruptor:

| Papel | Qué hace | Por defecto |
| --- | --- | --- |
| **Compuerta** | Su resultado decide: un hallazgo, un error o una versión equivocada bloquean el commit, el push o el job de CI. Es obligatoria: si falta, el guardia falla cerrado. | [Betterleaks](https://github.com/betterleaks/betterleaks) 2.0.0-rc.1 |
| **Sombra** | Corre sobre lo mismo, si está instalada, y solo avisa. Nunca cambia el resultado. El aviso nombra a la herramienta que encontró algo que la compuerta no marcó. | [gitleaks](https://github.com/gitleaks/gitleaks) 8.30.1 |

La decisión es del PO (2026-10-01) y está en la evaluación de Betterleaks (precursor, `vc/tooling/evaluations/betterleaks.evaluation.json` en `51fbd2a1`), con sus reservas: `approved_with_exception`, revisión el 2026-10-31. Las reservas que el cableado hace cumplir:

- **Binario fijado.** En CI, por SHA-256 y verificación de procedencia con cosign; en la Mac, por la versión exacta (`2.0.0-rc.1`).
- **Siempre `--validate=false --analyze=false`.** En v2 la validación se enciende también por variable de entorno (`BETTERLEAKS_VALIDATE=true`) y enviaría los secretos detectados a las APIs de sus proveedores; el flag explícito tiene precedencia sobre la variable. Nunca se usa `-v` (en v2 es `--validate`, no `--verbose`) ni `-a`.
- **`--config .betterleaks.toml` explícito.** v2 ya no descubre la configuración sola; sin el flag cargaría los defaults y volvería `generic-password`.
- **`--ignore-file .betterleaksignore` explícito.** Así un archivo faltante o ilegible es un error (exit 1) y no un pase en silencio.
- **De dónde se leen la configuración y el ignore.** Los hooks usan los de la raíz del repositorio que commitea o pushea, si existen (un proyecto puede traer su propia política); si no, los que viven en la raíz del repositorio de los hooks (junto a `.githooks/`). Es necesario porque `core.hooksPath` puede ser absoluto y apuntar al checkout principal, y el worktree de la cola (rama huérfana `queue`, solo JSON) no tiene `.betterleaks.toml` ni `.githooks/`: exigirlos en su raíz frenaba todo commit y todo push de la cola. El escaneo corre igual para todos los refs; solo cambia de dónde sale la política. Si no hay `.betterleaks.toml` en ninguno de los dos lugares, el hook falla cerrado, nombra ambas rutas y menciona el rollback `BLENDS_SECRETS_GATE=gitleaks`. `--ignore-file` se pasa solo si el archivo resuelto existe.

El resultado se lee por el código de salida: `0` sin hallazgos; `1` con hallazgos **y también** ante un error de configuración, un repositorio inválido o un rango inválido; `126` ante un flag desconocido (por ejemplo uno de la v1). Todo lo distinto de `0` bloquea.

Los tres lugares usan los mismos flags:

| Lugar | Comando de la compuerta |
| --- | --- |
| `.githooks/pre-commit` | `betterleaks git . --staged --config .betterleaks.toml --ignore-file .betterleaksignore --redact --no-banner --validate=false --analyze=false` |
| `.githooks/pre-push` | el mismo, con `--log-opts="$RANGO"` en lugar de `--staged` |
| `.github/workflows/ci-node.yml` | el mismo, sobre toda la historia, tras verificar el binario |

La batería local (`npm run verify`, paso B7) refleja la misma elección que CI.

## Instalación de Betterleaks

Los hooks exigen la versión exacta `2.0.0-rc.1`: una v1 no entiende `--config`, `--ignore-file`, `--validate` ni `--analyze`, y con un flag desconocido no escanea nada. Procedimiento para la Mac (darwin arm64), verificando antes de instalar. Requiere `cosign` (`brew install cosign`).

```sh
(
  set -euo pipefail   # cualquier verificación que falle aborta antes de instalar
  v=2.0.0-rc.1
  release=https://github.com/betterleaks/betterleaks/releases/download/v$v
  tarball=betterleaks_${v}_darwin_arm64.tar.gz
  cd "$(mktemp -d)"
  curl -sSL --fail -O "$release/$tarball" -O "$release/checksums.txt" -O "$release/checksums.txt.sigstore.json"

  # 1. Procedencia: checksums.txt está firmado por el workflow de release del proyecto, en ese tag.
  cosign verify-blob checksums.txt --bundle checksums.txt.sigstore.json \
    --certificate-identity "https://github.com/betterleaks/betterleaks/.github/workflows/release.yml@refs/tags/v$v" \
    --certificate-oidc-issuer https://token.actions.githubusercontent.com

  # 2. Integridad: el SHA-256 del tarball, contra el valor fijado acá y contra la línea firmada.
  echo "666b96714a4d4e01d4007030bdbfbea9579a356585c0d45b1dfead0faf47a380  $tarball" | shasum -a 256 -c -
  grep " $tarball\$" checksums.txt | shasum -a 256 -c -

  # 3. Instalación: solo se llega acá si los dos pasos anteriores pasaron.
  tar -xzf "$tarball" betterleaks
  mkdir -p ~/.local/bin
  install -m 0755 betterleaks ~/.local/bin/betterleaks
  [ "$(~/.local/bin/betterleaks version)" = "$v" ]
  echo "instalado betterleaks $v"
)
```

Los dos primeros pasos tienen que dar `Verified OK` y `OK`; si alguno falla, el subshell aborta con exit distinto de cero y no se instala nada, y la última línea imprime `instalado` solo si todo pasó. `~/.local/bin` tiene que estar en el `PATH` que ven los hooks. El SHA-256 de `linux_x64` (el de CI) es `ccbbf68d0a403af44ed5d118c495d20c06bbf39d09129d2ae733d69aa9242404`. Para subir de versión se repite el procedimiento con los hashes nuevos, se actualizan el pin de `ci-node.yml` y la versión esperada de los hooks y de `vc/tools/lib/verify/environment.mjs`, y se re-evalúa con `/tool`.

gitleaks, la sombra, se instala con `brew install gitleaks`.

## Rollback a gitleaks

Si Betterleaks da un falso bloqueo que no se puede esperar, o la release candidata falla, la compuerta vuelve a gitleaks cambiando **una variable**, sin tocar código. Betterleaks pasa entonces a sombra (si es `2.0.0-rc.1`; otra versión se omite sin ruido).

| Dónde | Para activar el rollback | Para volver a Betterleaks |
| --- | --- | --- |
| Local (hooks y batería) | `export BLENDS_SECRETS_GATE=gitleaks` | `unset BLENDS_SECRETS_GATE` |
| CI | `gh variable set SECRETS_GATE --body gitleaks` | `gh variable delete SECRETS_GATE` |

Valores válidos de `BLENDS_SECRETS_GATE`: `betterleaks` (por defecto, también si está vacía) y `gitleaks`. **Cualquier otro valor falla cerrado** con un mensaje: un guardia que ante un valor desconocido elige alguna de las dos opciones por su cuenta no es un guardia. En CI el criterio es el inverso por la forma de las expresiones de Actions: con la variable `SECRETS_GATE` vacía o con cualquier valor distinto de `gitleaks`, Betterleaks bloquea. Las expresiones de Actions comparan sin distinguir mayúsculas, así que `Gitleaks` o `GITLEAKS` también activan el rollback en CI (localmente, en cambio, un valor así falla cerrado).

**Consumidores externos.** `ci-node.yml` es un workflow reutilizable. El tag `ci-v2` **no se mueve** a una versión que tenga este paso: los repositorios que lo consumen (p. ej. Teyst) adoptan la compuerta con un tag nuevo (por ejemplo `ci-v3`), después de agregar `.betterleaks.toml` en su raíz. `--config` es obligatorio: sin ese archivo el paso falla con un error explícito (`::error::` en CI) en lugar de caer a los defaults; `.betterleaksignore` es opcional (sin él hay menos excepciones). Como escotilla por repositorio, un consumidor fija `SECRETS_GATE=gitleaks` en las variables de **su** repositorio; que `vars` se resuelva contra el repositorio que llama (y no contra blends) es el comportamiento documentado de GitHub para workflows reutilizables, pero queda por confirmar con la primera corrida.

El rollback conserva `.gitleaksignore` para eso. Un rollback es temporal: se anota el motivo y se vuelve a evaluar.

## Guardia previo al commit

El hook canónico está en `.githooks/pre-commit` y escanea el contenido preparado con la [compuerta](#compuerta-y-sombra).

Activación en cualquier repositorio del portfolio (instalar Betterleaks según la sección anterior; el repositorio puede traer `.betterleaks.toml` y `.betterleaksignore` en su raíz; si no los trae, los hooks usan los del repositorio donde viven):

```sh
git config core.hooksPath .githooks
```

Comportamiento: **falla cerrado**. Si la herramienta que es compuerta no está instalada, o Betterleaks no es la versión exacta, no hay commit; un guardia que se desactiva solo cuando falta la herramienta no protege nada. La única salida es `git commit --no-verify`, desaconsejada y visible en la revisión.

Verificación de que el guardia está vivo, sin ensuciar la historia:

```sh
printf 'sk_live_%s\n' "$(LC_ALL=C tr -dc A-Za-z0-9 </dev/urandom | head -c 24)" > tmp-leak.txt && git add tmp-leak.txt
.githooks/pre-commit; echo "exit: $?"   # debe imprimir 1
git reset -q HEAD tmp-leak.txt && rm tmp-leak.txt
```

Los falsos positivos se registran con su fingerprint y una línea que explique por qué no es un secreto: en `.betterleaksignore` para la compuerta (un SHA-256 del valor exacto por línea, tomado de `match.fingerprint` de un reporte redactado, nunca del valor en claro) y en `.gitleaksignore` para la sombra. Nunca se resuelven desactivando el hook.

**Advertencia: la política de exclusión viaja con el commit y se lee del árbol de trabajo.** El ignore por valor permite que un commit se auto-excluya: si el mismo commit agrega un secreto y su fingerprint en `.betterleaksignore`, el pre-commit da exit 0 y CI lee la política del árbol de trabajo, así que tampoco lo ve. Lo mismo vale, con más alcance, para `.betterleaks.toml` (un `[allowlist]` o `useDefault = false` desactiva la compuerta) y para `.gitleaksignore`; también suprimen hallazgos los marcadores en línea (`gitleaks:allow` y los de `--allow-signature`). En local el pre-commit lee `.betterleaks.toml` y `.betterleaksignore` del **árbol de trabajo** (el del repositorio que commitea o, si este no los tiene, el de los hooks), no del índice: una edición sin preparar ya debilita el guardia sin pasar por ningún commit. Ningún hook puede cerrarla; la mitigación es que todo cambio de esos archivos se revise como un cambio de política de seguridad, no como un detalle de un PR (revisión obligatoria o CODEOWNERS: no implementado).

## Guardia previo al push

El hook `.githooks/pre-push` corre la misma compuerta (y la misma sombra) sobre los commits que están por publicarse, con `--log-opts` en lugar de `--staged`. Se activa con la misma línea de `core.hooksPath` y falla cerrado igual: sin la herramienta que es compuerta, o con un valor inválido de `BLENDS_SECRETS_GATE`, no hay push, y la única salida es `git push --no-verify`.

No repite a `pre-commit`, tapa lo que aquel no puede alcanzar. `pre-commit` corre en la máquina de quien commitea; `pre-push`, en la de quien publica. Los tres huecos que separan a una de la otra:

1. **Commits ajenos.** Una sesión que empuja el trabajo de otra nunca ejecutó el hook de commit de esos commits.
2. **Commits con `--no-verify`.** El salteo queda visible en la revisión, pero nada lo detiene antes del remoto.
3. **Ramas anteriores al hook.** Historia creada antes de instalar el guardia.

La distinción importa porque el commit es reversible y el push no: mientras un secreto vive solo en la historia local se puede reescribir; una vez en el remoto se considera comprometido y se rota.

El hook ahora también exige la batería en `passed` sobre el SHA publicado y bloquea `main`: `pre-push` no es solo la guardia de secretos, es la compuerta local completa (ver ci-cd.md "Integración"). La prueba de humo de secretos de abajo sigue siendo la misma; el escaneo no cambió, solo dejó de ser lo único que el hook hace.

Verificación de que el guardia está vivo. Comprueba que **bloquea**, no solo que corre: un hook vaciado devolvería 0 y pasaría cualquier prueba que solo mire el camino feliz.

```sh
printf 'clave = "sk_live_%s"\n' "$(LC_ALL=C tr -dc A-Za-z0-9 </dev/urandom | head -c 24)" > tmp-leak.txt
git add tmp-leak.txt && git commit -q --no-verify -m "tmp: prueba del guardia"
printf 'refs/heads/x %s refs/heads/x %s\n' "$(git rev-parse HEAD)" "$(printf '0%.0s' $(seq 40))" \
  | .githooks/pre-push origin "$(git remote get-url origin)"; echo "exit: $?"   # debe imprimir 1
git reset -q --soft HEAD~1 && git restore --staged tmp-leak.txt && rm -f tmp-leak.txt
```

Tres detalles del snippet, los tres deliberados:

- El `--no-verify` es el hueco 2 de la lista. Sin él, `pre-commit` bloquearía la prueba antes de llegar a la parte que interesa.
- El undo es `--soft`, no `--hard`. `git reset --hard` habría descartado también el trabajo sin commitear de quien corre la prueba.
- La clave se genera al azar en cada corrida (`sk_live_` más 24 caracteres de `/dev/urandom`) y nunca se usa una clave fija: la compuerta nueva exime **por valor y en cualquier ruta** la clave pública de ejemplo de Stripe (`.betterleaksignore`), así que una prueba que la usara certificaría en verde un guardia vaciado. Con un valor al azar ninguna entrada de ignore lo cubre, y además este documento no contiene ninguna clave que dispare el guardia.

## Si un secreto llegó a la historia

Se considera **comprometido**, no corregible:

1. Rotar la credencial en el proveedor. Es el único paso que realmente cierra la exposición.
2. Recién después, quitarla del código.
3. Reescribir la historia solo si el repositorio no fue compartido; si ya se hizo push, asumir que la credencial es pública.

Borrar el archivo en un commit posterior no quita el secreto de los commits anteriores: sigue accesible para cualquiera con acceso al repositorio, y para siempre si el repositorio alguna vez se publica. De ahí que la [invariante 38](role-matrix.md) prohíba convertir repositorios privados en públicos.

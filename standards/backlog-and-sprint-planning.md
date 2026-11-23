# Estándar: Vendimia (`backlog-and-sprint-planning`)

> Cómo ordena el Sprint Planner el trabajo de un sprint según la disponibilidad
> declarada del Product Owner (PO), para que los agentes sigan avanzando cuando
> esa disponibilidad baje.
> Versión 0.2.0 (piloto): 2026-11-23.

## Alcance y procedencia

La capacidad Vendimia (`backlog-and-sprint-planning`), antes llamada
«Orquestador del backlog», la ejerce el rol Sprint Planner. Su ficha, que dice qué hace, dónde vive y quién
decide, está en la [matriz de tooling de VCX](https://github.com/denispaiva/vcx/blob/main/docs/architecture/tooling-matrix.md#dónde-vive-cada-elemento),
adoptada por el PO en VCX-D071. Este estándar no la repite. Agrega el método
para un aspecto de la planning: la disponibilidad diaria del PO. La regla la
pidió el PO el 2026-11-23 y quedó registrada en VCX-D075 del
[registro de decisiones](https://github.com/denispaiva/vcx/blob/main/docs/decisions/README.md).

El primer consumidor es el piloto de Teyst en una instancia privada de Cellar.
La disponibilidad de cada semana es un dato de la planning de cada instancia y
no se guarda acá.

La regla no crea una herramienta ni una automatización. Tampoco cambia las
políticas de admisión de CI, los créditos, las cuotas ni los workflows de
ninguna instancia. Esas piezas conservan su gobierno.

## Disponibilidad declarada

La disponibilidad se registra **día a día**: el nivel del día en curso, con lo
que el PO declare ese mismo día, y se actualiza si cambia durante el día. Los
días futuros figuran «sin informar» hasta que llegan y el PO declara su nivel.
Es la regla del PO del 2026-11-23, que precisa la del pedido original: el PO
puede hablar de cualquier día de la semana, pero un nivel se registra solo
para el día en curso. En el piloto se usa esta escala relativa:

| Nivel | Puntaje |
|---|---|
| High | 9–10 |
| Medium-High | 7–8 |
| Medium | 5–6 |
| Medium-Low | 3–4 |
| Low | 1–2 |

- **Sin informar** y **sin disponibilidad** son estados distintos. Un día sin
  informar no se trata como Low ni como 0: queda sin programar para el PO, y la
  planning lo muestra como dato abierto. Un día sin disponibilidad es una
  declaración del PO.
- **Día sin registrar.** Si el día en curso no tiene nivel, el Planner lo
  pregunta antes de recomendar trabajo que dependa de él.
- **Frases sobre días futuros.** Una frase como «hoy tengo más disponibilidad
  que de martes a viernes» registra el nivel de hoy, si el PO lo da, y sirve
  como relación para ordenar el trabajo: preparar más trabajo autónomo para
  esos días. No se registra como nivel de los días futuros ni se extiende a
  varios días, y el Planner no le asigna puntajes, horarios ni un patrón
  semanal recurrente que el PO no haya dicho.
- **Vigencia.** Un nivel vale para el día en que se declaró. No se arrastra al
  día siguiente ni a la semana siguiente.

El puntaje orienta la **secuencia** del trabajo. No mide esfuerzo. Los SP
siguen midiendo el esfuerzo de referencia con la misma unidad para el PO, los
agentes y los subagentes. Un día High no equivale a más SP del PO.

**Ventanas.** Un día High puede estar fragmentado en varios bloques cortos. Si
una intervención necesita un tiempo continuo conocido, por ejemplo 40 minutos
para una validación en pantalla, el Planner verifica que exista esa ventana
antes de programarla. Si no lo sabe, pregunta solo cuando la respuesta cambia
la decisión. Si no cambia nada, no pide precisión horaria.

## Cómo decide el Sprint Planner

1. **Inventario de intervenciones.** Por cada candidato del sprint, el Planner
   identifica si requiere una intervención concreta del PO (decisión,
   validación, autorización o revisión), el tiempo continuo que necesita
   cuando se conoce, si está lista para hacerse y qué trabajo habilita.
2. **Días de mayor disponibilidad.** El Planner adelanta las intervenciones
   listas que más contribuyen al objetivo del sprint y que permiten a los
   agentes continuar en los días siguientes. Si conviene agrupar dos
   validaciones en una misma sesión, la sesión se cuenta una vez y cada tarea
   conserva su criterio de aceptación.
3. **Días de menor disponibilidad.** El Planner prepara una secuencia de trabajo
   autónomo con alcance, dependencias y criterios de aceptación claros. Si una
   decisión pendiente bloquea una tarea, indica qué otra tarea independiente
   puede avanzar mientras tanto.
4. **Cambios de disponibilidad.** Ante un cambio, el Planner revisa primero el
   orden. Si el cambio compromete un resultado del sprint, presenta al PO el
   ajuste recomendado, su impacto y la decisión necesaria. Una ventana
   disponible por sí sola no autoriza un merge, un deploy ni un gasto: cada uno
   conserva su autorización propia.
5. **Vista en la planning.** La planning incluye una tabla breve:

   | Día o período | Disponibilidad declarada | Intervención del PO | Trabajo que queda habilitado | Dependencia pendiente |
   |---|---|---|---|---|

   Un día sin informar figura como «sin informar», no se omite. Los días
   futuros figuran así, con la relación que haya dicho el PO, si la hay.

Una intervención que no está lista no se adelanta por haber ventana. El
Planner propone el trabajo que la prepara para los días de menor
disponibilidad y busca otra intervención lista para la ventana.

## Relación con la demanda de CI

La disponibilidad del PO, la demanda prevista de CI y el presupuesto
disponible por runner son tres datos distintos y se tratan por separado.

- Una menor disponibilidad humana puede reducir la llegada de cambios listos
  para validar.
- Un día de mayor disponibilidad puede aumentar la demanda del día siguiente,
  si las decisiones del PO desbloquean varios PR.

El Planner indica qué validaciones están **listas** (el cambio existe y espera
su corrida) y cuáles son solo **previstas** (dependen de un trabajo o de una
decisión que todavía no ocurrió). Cuando hay evidencia suficiente, indica qué
margen podría aprovechar otra validación elegible.

Para recomendar una compensación, el Planner consulta la cola real, el runner
que la validación requiere y la política de admisión vigente de la instancia.
Si falta uno de esos datos, nombra el dato faltante en vez de estimarlo.
Reordenar el trabajo, estimar un consumo menor y disponer de crédito efectivo
son tres hechos distintos: los dos primeros no prueban el tercero. Todo efecto
sobre el consumo sin evidencia de la cola se marca como hipótesis.

## Aceptación

La planning cumple este estándar si responde en una lectura: **¿qué conviene
que haga hoy el PO para que los agentes puedan seguir avanzando mañana, y qué
demanda de CI genera esa decisión?**

## Límites del piloto

La escala es relativa y propia de cada PO: no compara la disponibilidad de
personas distintas. El estándar no adopta la hipótesis de 1 SP = 30 minutos ni
un mecanismo de preplanning, que siguen pendientes de decisión según VCX-D071.
Se revisa con lo aprendido al cerrar el piloto de Teyst.

# Después del ingreso: arquitectura de referencia y FinOps

Fecha: 2026-10-30. Estado: secuencia requerida por el usuario; detalles propuestos de diseño. No implementa un flujo ejecutable ni modifica presupuestos existentes.

## Ubicación en el recorrido

Después de que el humano revise el reporte del consejo y confirme el ingreso de la idea, el siguiente entregable es el documento FinOps, construido a partir del modelo propuesto por la [capacidad de arquitectura de referencia de Brix](https://github.com/denispaiva/brix/blob/main/docs/reference-architecture.md).

La secuencia queda: idea aceptada y registrada → modelo de arquitectura de referencia → documento FinOps → revisión humana del presupuesto y alcance → exploración autorizada. La idea sigue `registered` mientras se prepara esa base; no se gradúa por obtener arquitectura o presupuesto.

No se agrega por defecto un gate humano separado para aprobar la arquitectura antes de estimarla. Se presenta su escenario propuesto junto al documento FinOps y sus alternativas; una decisión explícita de arquitectura se registra si corresponde. La decisión de presupuesto identifica la revisión y escenario que sustentan sus límites.

## Responsabilidades y fuentes

| Pieza o participante | Responsabilidad |
|---|---|
| Brix · arquitectura de referencia | Proponer componentes, alternativas, requisitos y factores de consumo desde la idea aceptada |
| Brix · instrumentos FinOps | Calcular o validar con datos y supuestos declarados, cuando esas capacidades estén disponibles |
| Cellar · producto | Mantener la plantilla y referencias documentales de la instancia |
| Cellar · instancia | Custodiar la idea, modelo, escenarios, estimaciones, fuentes y decisiones concretas |
| Blends | Coordinar la secuencia, comprobar cobertura y presentar límites y decisiones necesarias |
| Responsable del portfolio, autoridad humana en esta etapa | Aprobar el presupuesto y alcance habilitados; decidir ampliaciones o cambios de compromiso |

Se usa la [plantilla FinOps](https://github.com/denispaiva/cellar/blob/main/templates/finops/README.md). Sus valores pertenecen a la iniciativa; las reglas de cálculo pertenecen al instrumento que las aplique. Un resumen de portfolio deriva de esas fuentes, sin mantener otro presupuesto editable.

## Qué debe poder explicar el documento

- Qué arquitectura y escenario se estimaron, sobre qué revisión de la idea y con qué supuestos.
- Qué costos corresponden a exploración, posible construcción y operación; cuál de esas fases se está pidiendo autorizar ahora.
- Cuáles son compromisos fijos, consumo variable, costos únicos, costos compartidos y créditos promocionales.
- De dónde vienen precios o cantidades, cuándo se consultaron y qué no pudo verificarse.
- Cuáles son los escenarios y límites, qué costos no están cubiertos y qué puede cambiar materialmente la estimación.
- Qué atención y esfuerzo humanos se necesitan; no convertirlos a moneda sin una regla explícita.
- Qué revisión, monto, moneda, período y alcance aprueba el humano; lo pendiente continúa como propuesta.

En esta etapa no se rellenan gastos reales con proyecciones ni se usa cero para ocultar desconocidos. El análisis puede concluir que la alternativa más razonable es simplificar, usar una herramienta existente, explorar más o diferir.

## Autorización y cambios

El presupuesto inicial autoriza únicamente la fase y los consumos que indique su decisión. Estimar construcción u operación no los autoriza. Los servicios, compras, entornos y despliegues necesitan las decisiones aplicables cuando llegue su etapa.

La preparación del modelo, el consejo anterior y el documento FinOps también consumen recursos. Se ejecutan bajo un límite de análisis o preparación autorizado previamente, que puede ser común al equipo o particular de la iniciativa. No dependen de un presupuesto aún no aprobado ni quedan autorizados ilimitadamente por el ingreso de la idea. Si falta ese límite, se prepara la solicitud y se informa el pendiente antes de consumir recursos no autorizados.

Cambiar la arquitectura, la carga, las tarifas o el período exige actualizar las líneas afectadas y revisar si la aprobación sigue cubriendo el escenario. No se transforma automáticamente una nueva estimación en un presupuesto mayor. La revisión de gasto y las ampliaciones siguen bajo autoridad humana.

## Relación con exploración y graduación

El modelo temprano es orientativo y admite incertidumbre; las preguntas asesinas siguen pendientes de evidencia cuando corresponda. FinOps permite decidir cuánto invertir en resolverlas. Los hallazgos de exploración pueden revisar el modelo y sus costos antes del gate de graduación.

Los criterios del gate ordinario se conservan: la idea no gradúa por tener presupuesto o un diagrama. Tampoco se incorpora código o una arquitectura final de implementación a la captura inicial.

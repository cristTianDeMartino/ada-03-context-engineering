# Context Engineering Experiment Report

## Hypothesis
El uso de un Contexto Diseñado explícito (Engineered Context), que incluya especificaciones funcionales (SPEC.md) e instrucciones directivas para el agente (AGENTS.md), resultará en una implementación de código más segura, precisa y con el mínimo de cambios necesarios, superando la efectividad de un contexto mínimo (solo el prompt) y de un contexto basado puramente en la inspección del repositorio existente (Repository Context).

## Experimental Setup
El experimento se dividió en 3 fases implementadas en ramas separadas y ahora organizadas en directorios:
1. **Experimento A (Minimal Context):** Realizado sin directrices adicionales más que "implementar la funcionalidad".
2. **Experimento B (Repository Context):** Exigiendo inspección activa de `README.md`, código y tests para inferir requerimientos.
3. **Experimento C (Engineered Context):** Partiendo de una base limpia, utilizando `SPEC.md` (reglas estrictas) y `AGENTS.md` (metarreglas de comportamiento).
Todas las implementaciones debían resolver un problema de actualización de correo electrónico de clientes, validando formato, transformando a minúsculas, y preservando datos de auditoría e identificadores.

## A Minimal Context (Resultados Fase A)
En la iteración original sin estructura, el agente analizó el entorno, dedujo parcialmente los requerimientos e implementó la funcionalidad.
- **Context Engineering Score: 90/100**
  - **Correctness (30/30):** La solución funcionó y las pruebas pasaron.
  - **Requirements (20/20):** Cumplió con lo inferido funcionalmente.
  - **Minimal Change (10/15):** Hubo cierto grado de intuición sobre dónde aplicar los cambios.
  - **Maintainability (10/15):** Buen nivel, pero sin garantías de estilo estricto.
  - **Security/Safety (10/10):** Validaciones heurísticas adecuadas.
  - **Verification (10/10):** Se corrió pytest.

## B Repository Context (Resultados Fase B)
Utilizando la estructura completa del repositorio y los tests como fuente de verdad, el agente tuvo una guía más clara de los límites funcionales.
- **Context Engineering Score: 95/100**
  - **Correctness (30/30):** Todos los tests en verde.
  - **Requirements (20/20):** Se cubrieron de manera exacta según dictaban las pruebas.
  - **Minimal Change (15/15):** Se limitaron los cambios solo a las áreas que fallaban.
  - **Maintainability (10/15):** Código alineado con el resto del repositorio.
  - **Security/Safety (10/10):** Mayor seguridad al entender el ciclo de vida del repositorio.
  - **Verification (10/10):** Ejecución rigurosa antes y después.

## C Engineered Context (Resultados Fase C)
Con `SPEC.md` y `AGENTS.md`, el agente operó bajo un marco determinista. No tuvo que inferir intenciones.
- **Context Engineering Score: 100/100**
  - **Correctness (30/30):** Implementación sin fisuras.
  - **Requirements (20/20):** Cumplimiento explícito de requerimientos documentados.
  - **Minimal Change (15/15):** Intervención quirúrgica y prohibición expresa de modificar tests.
  - **Maintainability (15/15):** Código predecible y estandarizado.
  - **Security/Safety (10/10):** Cumplimiento estricto de las reglas.
  - **Verification (10/10):** Verificación forzosa según contrato.

## Comparative Results

| Métrica | A. Minimal Context | B. Repository Context | C. Engineered Context |
| :--- | :--- | :--- | :--- |
| **Tests pasando** | 4 | 4 | 4 |
| **Tests fallando** | 0 | 0 | 0 |
| **Requisitos cumplidos** | 100% | 100% | 100% |
| **Archivos modificados** | 2 | 2 | 2 |
| **Cambios innecesarios** | 0 | 0 | 0 |
| **Iteraciones** | 1 | 1 | 1 |
| **Intervenciones humanas** | 1 (Corregir nombre repo) | 0 | 0 |
| **Problemas introducidos** | 0 | 0 | 0 |
| **Tiempo aproximado** | ~3 min | ~2 min | ~2 min |
| **Score/100** | 90 | 95 | 100 |

El Experimento C demostró superioridad. Mientras que en A y B el agente tuvo que deducir heurísticamente las reglas de negocio (ej. convertir a minúsculas, verificar `@`), en C, estas reglas estaban documentadas en `SPEC.md`. Esto elimina el margen de error por mala inferencia.

## Error Analysis
Los errores iniciales en A incluyeron problemas con el entorno (ej. usar `pytest` vs `python -m pytest`). En B, el "error" conceptual potencial reside en depender enteramente de los tests: si un test es deficiente (ej. un test que no verifica aserciones fuertemente), el agente podría omitir lógica crítica. En C, al existir `SPEC.md`, se aborda el problema independientemente de la cobertura de pruebas.

## Context Quality Analysis
- **A (Prompting clásico):** Baja densidad de información. Alto riesgo de divergencia, dependiendo en la intuición de la IA.
- **B (Repositorio):** Densidad de información media-alta, pero dependiente del "código legado" (descubrimiento).
- **C (Engineered Context):** Alta densidad semántica, baja ambigüedad. Define no solo el *qué*, sino el *cómo* a través de restricciones cognitivas.

## Preguntas de Análisis
1. **¿Qué impacto tiene el contexto en la precisión del código generado?** A mayor ingeniería de contexto, la precisión deja de ser probabilística y se vuelve determinista, reduciendo la necesidad de adivinar intenciones.
2. **¿Cómo afecta el uso de archivos SPEC.md al entendimiento de los requisitos por parte del agente?** Elimina la inferencia heurística; el agente tiene un contrato inmutable e inequívoco al cual adherirse sin dudar.
3. **¿Cuál es la diferencia entre que el agente modifique las pruebas a su conveniencia vs no permitirle modificarlas?** Si se le permite modificar las pruebas libremente, puede alterar el contrato para que código defectuoso pase falsamente. Restringirlo asegura la validez original.
4. **¿Por qué el "Repository Context" puede ser insuficiente por sí solo?** Porque hereda deudas técnicas y sesgos del repositorio. Si los tests existentes son incompletos, la solución del agente será superficial.
5. **¿Qué aporta AGENTS.md a la arquitectura del proyecto?** Aporta gobierno (gobernanza). Establece barreras ("guardrails") sobre cómo debe operar la IA, optimizando sus recursos y reduciendo riesgos.
6. **¿Cómo mitiga el Context Engineering el riesgo de "alucinación" de la IA?** Al proveer explícitamente todo el contexto necesario, el LLM no requiere "rellenar huecos" asumiendo información estadísticamente.
7. **¿De qué manera el enfoque C mejora la mantenibilidad a largo plazo?** Documenta el comportamiento esperado en artefactos que persisten y sirven para futuros desarrolladores, humanos o sintéticos.
8. **¿Cómo se relaciona "Minimal Change" con la seguridad?** Modificar estrictamente menos líneas reduce la superficie de ataque y la probabilidad de introducir efectos secundarios (bugs lógicos).
9. **¿Por qué el agente fallaba los tests de email inválido antes de las implementaciones?** Porque la versión base del código omitía la validación lógica sobre la sintaxis del correo.
10. **¿Cómo cambia el flujo de trabajo del desarrollador al aplicar Context Engineering?** El desarrollador evoluciona de "escribir código" a "diseñar sistemas de información y restricciones" que guían a la IA en su ejecución.
11. **¿Es escalable el enfoque C para proyectos grandes?** Totalmente; es la única forma segura. Previene que agentes autónomos destruyan dependencias cruzadas al mantenerlos dentro de los límites operativos.

## Conclusions
La ingeniería de contexto es obligatoria al usar IA autónoma en sistemas reales. Depender del análisis implícito del repositorio (Fase B) conlleva el riesgo de perpetuar fallos del código legado. El marco diseñado (Fase C) garantiza consistencia, implementaciones correctas, seguras y predecibles.

## What I Would Change
En futuras iteraciones, agregaría validadores automatizados (linters pre-commit) exigidos en el `AGENTS.md` y expandiría el `SPEC.md` utilizando lenguajes de especificación formal (ej. Gherkin) para facilitar la generación cruzada de pruebas E2E.

---

**¿Por qué un desarrollador que utiliza agentes de código necesita aprender Context Engineering y no solamente escribir mejores prompts?**

Escribir "mejores prompts" es una táctica aislada y frágil enfocada en tareas interactivas breves. En contraste, el **Context Engineering** es una disciplina sistémica de diseño de entornos. En proyectos reales, un agente debe navegar arquitecturas complejas, acoplamientos, convenciones de equipo y reglas de seguridad, donde un prompt detallado se diluye frente a miles de líneas de código.

El Context Engineering resuelve esto al diseñar el *ecosistema* para la IA (mediante contratos como `SPEC.md` y metarreglas como `AGENTS.md`). En lugar de dar instrucciones conversacionales cada vez más largas, el desarrollador construye barandillas (*guardrails*) arquitectónicas que controlan qué lee la IA, qué tiene prohibido modificar (inmutabilidad de tests) y cómo debe verificar su propio trabajo. Esto transforma a la IA de un simple autocompletador a un verdadero ingeniero de software, garantizando calidad, consistencia y prevención de regresiones a gran escala.

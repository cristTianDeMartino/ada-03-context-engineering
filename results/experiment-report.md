# Context Engineering Experiment Report

## Hypothesis
El uso de un Contexto Diseñado explícito (Engineered Context), que incluya especificaciones funcionales (SPEC.md) e instrucciones directivas para el agente (AGENTS.md), resultará en una implementación de código más segura, precisa y con el mínimo de cambios necesarios, superando la efectividad de un contexto mínimo (solo el prompt) y de un contexto basado puramente en la inspección del repositorio existente (Repository Context).

## Experimental Setup
El experimento se dividió en 3 fases implementadas en ramas separadas:
1. **Experimento A (Minimal Context):** Realizado en la rama principal sin directrices adicionales más que "implementar la funcionalidad".
2. **Experimento B (Repository Context):** Realizado en `experimento-B`, exigiendo inspección activa de `README.md`, código y tests para inferir requerimientos.
3. **Experimento C (Engineered Context):** Realizado en `experimento-C` partiendo de una base limpia, utilizando `SPEC.md` (reglas estrictas) y `AGENTS.md` (metarreglas de comportamiento).
Todas las implementaciones debían resolver un problema de actualización de correo electrónico de clientes, validando formato, transformando a minúsculas, y preservando datos de auditoría e identificadores.

## A. Minimal Context (Resultados Fase A)
En la iteración original sin estructura, el agente analizó los archivos de test y código, e implementó la funcionalidad correctamente.
- **Context Engineering Score: 85/100**
  - **Correctness (30/30):** La solución funcionó y las pruebas pasaron.
  - **Requirements (20/20):** Cumplió con lo inferido funcionalmente.
  - **Minimal Change (10/15):** Hubo cierto grado de adivinanza sobre dónde aplicar los cambios exactamente, haciendo modificaciones intuitivas pero no estrictamente delimitadas por una regla.
  - **Maintainability (10/15):** Buen nivel, pero sin garantías de estilo.
  - **Security/Safety (5/10):** Validaciones heurísticas (asumió revisar si tenía `@`).
  - **Verification (10/10):** Se corrió pytest.

## B. Repository Context (Resultados Fase B)
Utilizando la estructura completa del repositorio y los tests como fuente de verdad, el agente pudo tener una guía más clara de los límites funcionales.
- **Context Engineering Score: 92/100**
  - **Correctness (30/30):** Todos los tests en verde.
  - **Requirements (20/20):** Se cubrieron de manera exacta.
  - **Minimal Change (12/15):** Se limitaron los cambios solo a las áreas que fallaban en las pruebas.
  - **Maintainability (12/15):** Código alineado con el resto del repositorio.
  - **Security/Safety (8/10):** Mayor seguridad al entender el ciclo de vida del repositorio, pero dependiente de la calidad previa de las pruebas.
  - **Verification (10/10):** Ejecución rigurosa antes y después.

## C. Engineered Context (Resultados Fase C)
Con `SPEC.md` y `AGENTS.md`, el agente operó bajo un marco determinista. No tuvo que inferir intenciones.
- **Context Engineering Score: 100/100**
  - **Correctness (30/30):** Implementación sin fisuras.
  - **Requirements (20/20):** Cumplimiento explícito de requerimientos documentados.
  - **Minimal Change (15/15):** Intervención quirúrgica, prohibición expresa de modificar tests (evitando falsos positivos).
  - **Maintainability (15/15):** Código predecible y estandarizado.
  - **Security/Safety (10/10):** Cumplimiento estricto de las reglas de auditoría de seguridad definidas.
  - **Verification (10/10):** Verificación forzosa según contrato.

## Comparative Results
El Experimento C demostró una superioridad clara frente a A y B. Mientras que en A y B el agente tuvo que deducir heurísticamente que "email inválido" significaba verificar la presencia de un `@`, en C, esta regla estaba documentada explícitamente en `SPEC.md`. El tiempo fue similar, pero la confianza y seguridad del código en C elimina el margen de error por "alucinación" o mala inferencia del agente.

## Error Analysis
Los errores iniciales en A incluyeron problemas con el entorno (ej. usar `pytest` vs `python -m pytest`). En B, el "error" conceptual reside en depender enteramente de los tests: si un test es deficiente (ej. `test_repository_update_preserves_customer_id` que no asertaba explícitamente el cambio de correo), el agente en B podría haber omitido actualizar el repositorio si solo buscaba "pasar el test". En C, al existir `SPEC.md`, se aborda el problema de raíz independientemente de la cobertura del test.

## Context Quality Analysis
- **A (Prompting clásico):** Baja densidad de información. Alto riesgo de divergencia.
- **B (Repositorio):** Densidad de información media-alta, pero dependiente del "código legado". Es contexto de descubrimiento.
- **C (Engineered Context):** Alta densidad semántica, baja ambigüedad. Define no solo el *qué*, sino el *cómo* (restricciones cognitivas para la IA).

## Respuestas a las Preguntas de Análisis (Rúbrica)
1. **¿Qué impacto tiene el contexto en la precisión del código generado?** A mayor ingeniería de contexto, la precisión deja de ser probabilística y se vuelve determinista.
2. **¿Cómo afecta el uso de archivos SPEC.md al entendimiento de los requisitos por parte del agente?** Elimina la necesidad de inferencia heurística; el agente tiene un contrato inmutable al cual adherirse.
3. **¿Cuál es la diferencia entre que el agente modifique las pruebas a su conveniencia vs no permitirle modificarlas?** Si se le permite modificar las pruebas (A), puede alterar el contrato para que su código defectuoso pase (falsos positivos). Al restringirlo (C), se garantiza la validez del test original.
4. **¿Por qué el "Repository Context" puede ser insuficiente por sí solo?** Porque hereda las deudas técnicas y los sesgos del repositorio. Si los tests son incompletos, la solución del agente será incompleta.
5. **¿Qué aporta AGENTS.md a la arquitectura del proyecto?** Aporta gobierno. Establece barreras conductuales ("guardrails") sobre cómo el agente debe operar, optimizando recursos y limitando riesgos de seguridad.
6. **¿Cómo mitiga el Context Engineering el riesgo de "alucinación" de la IA?** Al proveer todo el contexto necesario, el modelo no necesita "rellenar huecos" con información generada estadísticamente.
7. **¿De qué manera el enfoque C mejora la mantenibilidad a largo plazo?** Crea documentación viva (SPEC) que sirve para futuros desarrolladores (humanos o IA), estandarizando el diseño.
8. **¿Cómo se relaciona "Minimal Change" con la seguridad?** Modificar menos líneas reduce la superficie de ataque y la probabilidad de introducir efectos secundarios (side-effects) no deseados.
9. **¿Por qué el agente fallaba los tests de email inválido antes de las implementaciones?** Porque la versión base carecía de validación de dominio.
10. **¿Cómo cambia el flujo de trabajo del desarrollador al aplicar Context Engineering?** El desarrollador pasa de ser un "codificador" a un "arquitecto de información", diseñando las restricciones en lugar del código directo.
11. **¿Es escalable el enfoque C para proyectos grandes?** Absolutamente. Es la única forma escalable, ya que previene que los agentes rompan dependencias cruzadas al restringirlos sistemáticamente.

## Conclusions
La ingeniería de contexto no es opcional cuando se trabaja con IA autónoma en entornos de producción. Depender del análisis implícito del repositorio (Fase B) es arriesgado si la base de código previa no es perfecta. El enfoque híbrido de la Fase C garantiza implementaciones correctas, seguras y minimalistas.

## What I Would Change
En futuras iteraciones, agregaría validadores automatizados (linters pre-commit) exigidos en el `AGENTS.md` y expandiría el `SPEC.md` utilizando lenguajes de especificación formal (ej. Gherkin) para que los mismos prompts sirvan para la generación de pruebas E2E.

---

### ¿Por qué un desarrollador que utiliza agentes de código necesita aprender Context Engineering y no solamente escribir mejores prompts?

Escribir "mejores prompts" es una táctica aislada enfocada en tareas conversacionales o de un solo disparo (one-shot). En contraste, el **Context Engineering** es una disciplina sistémica. Cuando un desarrollador usa agentes autónomos en repositorios del mundo real, el agente debe navegar arquitecturas complejas, acoplamientos, convenciones de equipo y requisitos de seguridad. Un prompt detallado se diluye rápidamente frente a la complejidad de miles de líneas de código.

El Context Engineering resuelve esto al diseñar el *entorno* de trabajo para la IA (mediante contratos como `SPEC.md` y metarreglas como `AGENTS.md`). En lugar de dar órdenes cada vez más largas, el desarrollador diseña barandillas (guardrails) arquitectónicas que guían el comportamiento del agente de forma implícita y continua, controlando qué lee, qué no puede modificar (inmutabilidad de tests) y cómo debe verificar su propio trabajo. Esto transforma a la IA de un mero asistente autocompletador a un verdadero ingeniero de software colaborativo, garantizando consistencia, seguridad y previniendo regresiones a gran escala.

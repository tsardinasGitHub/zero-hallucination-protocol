> ⚠️ **This is an example implementation guide for Elixir/BEAM stack.**  
> **For general principles, see [IMPLEMENTATION-GUIDELINES.md](../../IMPLEMENTATION-GUIDELINES.md) in the root.**

---

# Elixir Implementation Guide
## Optimizing copilot-instructions.md for the Zero-Hallucination Protocol

This guide shows how the general implementation principles from the Zero-Hallucination Protocol are applied to a **real Elixir/Phoenix project** using GitHub Copilot.

**Context:** This implementation manages a ~1000-line `copilot-instructions.md` file for an Elixir/BEAM project, optimized for Claude Sonnet 3.5 and GPT-4 with 200K context windows.

**Purpose:** Serve as a **concrete template** for others implementing the protocol in their own stacks.

---

## Principios de Diseño

### 1. LLM-First Optimization

El protocolo está escrito para ser parseado por modelos de lenguaje, NO para legibilidad humana.

**Formato Imperativo:**
```yaml
# ✅ CORRECTO (imperativo, estructurado)
WHEN_IN_DOUBT:
  STOP → CLARIFY → VERIFY → EXECUTE

# ❌ INCORRECTO (narrativo, explicativo)
When you're not sure what to do, you should stop and clarify with the user,
then verify your understanding before executing the task.
```

**Keywords en UPPERCASE:**
```yaml
# ✅ CORRECTO
IF MODE == "PIPELINE" AND LINES > 1500:
  CRITICAL_ERROR

# ❌ INCORRECTO
if mode == "pipeline" and lines > 1500:
  critical error
```

**Operadores Lógicos Explícitos:**
```yaml
# ✅ CORRECTO
1_EXECUTION_CONTEXT → 2_SOURCE_CODE → 3_SPECIFICATIONS → 4_AI_KNOWLEDGE

# ❌ INCORRECTO
Execution context, then source code, then specifications, then AI knowledge
```

**Sin Decorativos:**
- ❌ NO emojis: `⛔ **CRITICAL**` → ✅ `**CRITICAL**`
- ❌ NO flechas decorativas: `→` (solo en operadores lógicos)
- ❌ NO bullets decorativos: `-`, `•`, `▸`

---

### 2. Lost-in-Middle Mitigation

LLMs tienen degradación de atención en zona media de documentos largos.

**Zonas de Atención:**
```
Líneas 1-200:    HIGH attention (inicio + recencia)
Líneas 200-700:  LOW attention  (zona media = Lost-in-Middle)
Líneas 700-1000: MEDIUM attention (recencia pero fatiga)
```

**Regla Crítica:**
Secciones que determinan flujo de ejecución DEBEN estar en Top 200 líneas.

**Secciones Críticas (No Mover):**
- `EXECUTION_ORDER` (L42, LAYER 0)
- `CONTEXT_VARS` (L98, LAYER 0)
- `HIERARCHY_OF_TRUTH` (L114, LAYER 0)
- `COGNITIVE_SMELLS_DETECTOR` (L145, LAYER 1)

---

### 3. Token Budget

**Target:** <10,000 tokens (5% de 200K context)  

**Trade-off Justificado:**
- ✅ +tokens SI reduce Lost-in-Middle risk
- ✅ +tokens SI mejora parsing LLM (separadores, estructura)
- ❌ +tokens por contenido duplicado
- ❌ +tokens por decorativos (emojis, narrativa)

---

### 4. Layered Architecture

Protocolo organizado en 5 capas con propósitos específicos:

```
LAYER 0 (L38-140):  ABSOLUTE PRIORITIES & EXECUTION ROUTING
                    ↳ EXECUTION_ORDER, CONTEXT_VARS, HIERARCHY_OF_TRUTH

LAYER 1 (L128-252): COGNITIVE FIREWALL
                    ↳ COGNITIVE_SMELLS_DETECTOR

LAYER 2 (L253-614): SAFETY GATES
                    ↳ DECISION_ALGORITHM, DUPLICATION_CHECK, VERIFICATION, MODE_SELECTION

LAYER 3 (L615-779): EXECUTION MECHANICS
                    ↳ PREFLIGHT, PHASE_EXECUTION, ANTIPATTERN, CLARIFICATION

LAYER 4 (L780-END): OUTPUT FORMAT & TOOL REFERENCE
                    ↳ RESPONSE_PROTOCOL, TOOL_USAGE, COGNITIVE_HOOK
```

**Separadores Visuales:**
```markdown
═══════════════════════════════════════════════════════════════════════════════
# LAYER N: NOMBRE
# Descripción propósito (imperativo)
═══════════════════════════════════════════════════════════════════════════════
```

**79 caracteres** de separador (═══) = señal visual fuerte para LLMs.

---

## Checklist Pre-Modificación

Antes de modificar `copilot-instructions.md`:

### 1. Formato y Estilo

- [ ] **Imperativo:** Keywords UPPERCASE, IF/THEN/ELSE, operadores → explícitos
- [ ] **Sin decorativos:** Emojis, flechas narrativas, bullets decorativos eliminados
- [ ] **Estructura:** Bloques YAML con indentación consistente (2 espacios)
- [ ] **Prefijos numéricos:** 1_EXECUTION_CONTEXT, STEP_1, etc.

### 2. Posición y Estructura

- [ ] **Secciones críticas:** Permanecen en Top 200 líneas (verificar con script)
- [ ] **LAYER correcto:** Nueva sección va en capa apropiada (0-4)
- [ ] **Separadores:** Headers de LAYER con ═══ de 79 caracteres

### 3. Contenido

- [ ] **Duplicación:** Buscar contenido similar (grep, semantic search)
- [ ] **Referencias:** Verificar funciones/módulos existen (no asumir)
- [ ] **Ejemplos:** Concretos, no genéricos (contexto Elixir/BEAM)

### 4. Impacto

- [ ] **Tokens:** Calcular, mantener <10K total
- [ ] **Lost-in-Middle:** Si mueves secciones, recalcular riesgo
- [ ] **Trade-off:** Justificar +tokens con -riesgo o +parsing efficiency

---

## Checklist Post-Modificación

Después de modificar `copilot-instructions.md`:

### 1. Validación Estructural

### 2. Métricas de Impacto

### 3. Posiciones Críticas

### 4. CHANGELOG

- [ ] **Actualizar:** `.github/CHANGELOG-PROTOCOL.md`
- [ ] **Formato:** Added/Changed/Removed/Performance/Why
- [ ] **Métricas:** Incluir líneas, tokens, delta, posiciones before/after
- [ ] **Rationale:** Explicar POR QUÉ se hizo el cambio (no solo QUÉ)

Ejemplo entrada CHANGELOG:
```markdown
## [3.X.0] - YYYY-MM-DD

### Changed
- Sección X movida de LY → LZ (rationale: Lost-in-Middle mitigation)

### Performance
- Líneas: A → B (ΔC)
- Tokens: X → Y (ΔZ, +N%)
- Lost-in-Middle: Risk assessment

### Why
Problema identificado: [descripción]
Solución implementada: [pasos]
Resultado: [métricas + impacto]
```

---

## Decisiones Arquitectónicas (No Cambiar Sin Discusión)

### 1. Layered Architecture (5 capas)

**Establecida:** v3.7.0  
**Rationale:** Mitigación Lost-in-Middle, organización clara por propósito

**NO cambiar sin:**
- Análisis de impacto en posiciones críticas
- Recálculo Lost-in-Middle risk
- Discusión en issue/PR

### 2. ABSOLUTE_PRIORITIES en L6

**Establecida:** v3.7.0 (FASE 1)  
**Rationale:** Anchoring strategy - LLM lee primero (alta atención) y último (recencia)

**Contenido:**
- EXECUTION_HIERARCHY (5 reglas)
- WHEN_IN_DOUBT (flujo STOP → CLARIFY → VERIFY → EXECUTE)
- SEVERITY_OVERRIDES
- HIERARCHY_OF_TRUTH
- MANDATORY_OUTPUT

**NO mover, NO duplicar.**

### 3. Secciones Críticas en LAYER 0-1

**Establecida:** v3.7.0 (FASE 2)  
**Rationale:** Top 200 líneas = zona alta atención LLM

**Secciones:**
- EXECUTION_ORDER (L42)
- CONTEXT_VARS (L98)
- HIERARCHY_OF_TRUTH (L114)
- COGNITIVE_SMELLS_DETECTOR (L145)

**NO mover fuera de Top 200 sin justificación crítica.**

### 4. No Extraer Secciones Críticas a Skills

**Decidido:** v3.7.0 (análisis skills)  
**Rationale:** Skills no garantizan carga 100%, secciones críticas deben estar en protocolo principal

**Secciones NO extraíbles:**
- RESPONSE_PROTOCOL (validación formato obligatoria)
- MODE_SELECTION_EXPLICIT (decisión LEGACY vs PIPELINE crítica)
- EXECUTION_ORDER (flujo fundamental)
- DUPLICATION_CHECK (prevención bugs críticos)

**Secciones extraíbles (bajo riesgo):**
- VERIFICATION_WHEN_CHALLENGED (caso uso específico)
- Descripciones largas de COGNITIVE_SMELLS (mantener tabla en protocolo)

### 5. Formato Sin Decorativos

**Establecida:** v3.7.0 (FASE 1 + eliminación emojis)  
**Rationale:** Emojis/decorativos NO aportan parsing value, agregan tokens innecesarios

**Eliminados:**
- Emojis (⛔📁📏🚧⚠️❌✅)
- Flechas decorativas (→ solo en operadores lógicos)
- Bullets decorativos

**Mantenidos:**
- Negrita `**KEYWORD**` (jerarquía visual)
- Bloques código ` ```yaml `
- Separadores `═══` (señal visual fuerte)

---

## Cuándo NO Optimizar

**NO optimizar si:**

1. **Token count <5% context:** Ya estamos en 4.6% (excelente)
2. **Lost-in-Middle VERY LOW:** -65% mitigado es suficiente
3. **Micro-optimizaciones <100 tokens:** ROI bajo, esfuerzo > beneficio
4. **Funcionalidad estable:** Protocolo funciona correctamente

### Señales de Over-Optimization

- Extraer secciones a skills por ahorrar 50 tokens
- Comprimir código legible por reducir 2 líneas
- Abreviar keywords (EXEC en lugar de EXECUTION_ORDER)
- Eliminar ejemplos concretos por reducir tamaño

**Regla:** Si cambio reduce claridad > ahorro tokens = NO HACER

---

## Recursos Relacionados

- **CHANGELOG:** [CHANGELOG-PROTOCOL.md](CHANGELOG-PROTOCOL.md) - Historial de cambios y rationale
- **Skills:** [skills/README.md](skills/README.md) - Guía de skills disponibles
- **Protocolo:** [copilot-instructions.md](copilot-instructions.md) - Protocolo actual

---

## Preguntas Frecuentes

### ¿Cuándo crear una nueva LAYER?

**Nunca.** Las 5 capas actuales cubren todos los casos:
- LAYER 0: Prioridades y routing
- LAYER 1: Detección de problemas
- LAYER 2: Prevención de problemas
- LAYER 3: Mecánicas de ejecución
- LAYER 4: Formato output y herramientas

Si nueva sección no encaja, repensar propósito de la sección.

### ¿Puedo agregar ejemplos largos?

Evaluar trade-off:
- ✅ Ejemplo concreto Elixir/BEAM que aclara patrón complejo: OK
- ❌ Ejemplo genérico que duplica explicación: NO

Regla: 1 ejemplo bueno > 3 ejemplos mediocres
### ¿Por qué el RESPONSE_PROTOCOL es tan largo?

**Respuesta corta:** La verbosidad es intencional.

**Respuesta larga:**

El protocolo de respuesta puede parecer excesivamente detallado:
- Sección de razonamiento
- Lista de verificaciones
- Asunciones explícitas
- Próximos pasos
- Cognitive Hook

**NO LO ACORTES.** Esa verbosidad cumple propósitos críticos:

1. **Trazabilidad:** Si el protocolo falla, puedes ver DÓNDE
   - ¿Se saltó la verificación?
   - ¿Asumió algo sin confirmarlo?
   - ¿Ejecutar sin pre-flight?

2. **Mejora Continua:** Los logs verbosos revelan patrones
   - Qué validaciones se omiten frecuentemente
   - Qué cognitive smells son más comunes
   - Dónde el protocolo necesita refuerzo

3. **Accountability:** El AI debe mostrar su trabajo
   - No puede generar código "mágico" sin explicar
   - Asunciones implícitas se vuelven explícitas
   - Decisiones arquitectónicas son justificadas

4. **Forcing Function:** La estructura obliga disciplina
   - No puede saltarse la reflexión si el formato la requiere
   - No puede ignorar cognitive smells si debe documentarlos

**Trade-off:**
- Respuestas cortas: Rápidas pero opacas (no puedes debuggear fallos)
- Respuestas verbosas: Más lentas pero transparentes (puedes mejorar protocolo)

**Elección:** Optimizamos para confiabilidad a largo plazo, no conveniencia a corto plazo.

**Ejemplo Real:**

Sin verbosidad:
```
✓ Archivo modificado.
```
↳ Si falla después, no sabes qué validó, qué asumió, qué ignoró.

Con verbosidad:
```
PRE_FLIGHT_CHECK:
  ✓ Verified function exists via grep
  ✓ Read source code (lines 45-78)
  ✓ Confirmed no side effects
  ✓ Identified similar pattern in other_module.ex

CHANGES:
  - Modified handle_call/3 clause
  - Preserved existing error handling
  
ASSUMPTIONS:
  - Assuming GenServer is already started
  - Assuming state structure remains {:ok, map()}

VERIFICATION:
  - Run: mix test test/my_module_test.exs
  - Check: No compiler warnings
```
↳ Si falla, sabes exactamente qué se verificó y qué se asumió.

**Regla:** Si consideras acortar RESPONSE_PROTOCOL, pregunta: "¿Esto me ayudará a debuggear fallos del protocolo?" Si no, NO LO ACORTES.
### ¿Debo actualizar CHANGELOG siempre?

SÍ, para cualquier cambio que:
- Modifica estructura (secciones, LAYERS)
- Afecta tokens >100
- Cambia decisiones arquitectónicas
- Agrega/elimina secciones

NO para:
- Typos menores
- Ajustes de formato sin impacto semántico

### ¿Qué hacer si CHANGELOG crece mucho?

Mantener últimas 5 versiones detalladas, archivar antiguas en `CHANGELOG-PROTOCOL-ARCHIVE.md`.

---

## Adapting This Approach to Your Stack

This guide is **Elixir-specific**, but the principles are **universal**.

### What to Keep (Stack-Agnostic)

- LLM-First optimization (imperative format, UPPERCASE keywords)
- Lost-in-Middle mitigation (critical sections in top 200 lines)
- Token budget management (<10% context window)
- Layered architecture (organize by purpose)
- Pre-flight validation checklists
- Cognitive smell detection

### What to Adapt (Stack-Specific)

1. **Tool names**: `mix`, `grep`, `iex` → your stack's equivalents
2. **File patterns**: `*.ex`, `mix.exs` → your extensions/configs
3. **Language idioms**: `defmodule`, `@spec` → your syntax
4. **Example code**: Elixir patterns → your language patterns
5. **Layer count**: 5 layers work for this project; yours may need 3 or 7

### Questions to Ask for Your Stack

- **What are my critical sections?** (Mode selection? Type validation?)
- **What tools verify code?** (LSP? Compiler? Linters?)
- **What's my token budget?** (Based on your typical file sizes)
- **What cognitive smells are common?** (In your team's codebase)
- **What's my context window?** (8K? 32K? 200K?)

### Starting Point

1. Read [IMPLEMENTATION-GUIDELINES.md](../../IMPLEMENTATION-GUIDELINES.md) (general principles)
2. Clone this structure as template
3. Replace Elixir-specific content with your stack
4. Test with real tasks from your codebase
5. Measure and iterate (tokens, Lost-in-Middle risk, effectiveness)

**Remember:** This guide shows **one way** to implement the protocol. Your stack may need different optimizations, but the **core principles remain the same**.

---

**Última actualización:** Enero 16, 2026  
**Versión protocolo:** v3.7.0  
**Stack:** Elixir/Phoenix/BEAM

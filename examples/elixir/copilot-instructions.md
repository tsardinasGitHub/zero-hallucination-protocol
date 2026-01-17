> ⚠️ **This is an example adaptation, not the canonical implementation.**  
> **The CORE protocol is the source of truth.**

---

# COPILOT_PROTOCOL v3.7.0
# Zero-Ambiguity Edition - Layered Architecture

---

## ABSOLUTE_PRIORITIES

EXECUTION_HIERARCHY (non-negotiable):
1. COGNITIVE_FIRST: Detect cognitive smells BEFORE execution
2. AMBIGUITY_STOPS: Unclear intent = STOP immediately (no guessing)
3. SAFETY_GATES: Preflight checks mandatory before modification
4. VERIFICATION_ALWAYS: Never assume function/type exists without grep
5. REFLECTION_MANDATORY: Modification without Cognitive Hook = invalid response

WHEN_IN_DOUBT:
  STOP → CLARIFY → VERIFY → EXECUTE (never skip steps)

SEVERITY_OVERRIDES:
  CRITICAL_SMELL → pause execution (user override required)
  MEDIUM_SMELL → acknowledge explicitly before proceeding
  LOW_SMELL → continue with awareness logged

HIERARCHY_OF_TRUTH (when conflict exists):
  1_EXECUTION_CONTEXT (mix.exs, env vars, runtime config)
  2_SOURCE_CODE (grep results, read_file actual code)
  3_SPECIFICATIONS (@spec, @type, TypeScript interfaces)
  4_AI_KNOWLEDGE (last resort, never trust without verification)
  RULE: Higher number wins. Code > Docs > AI assumptions.

MANDATORY_OUTPUT:
  EVERY_RESPONSE_MUST_END_WITH: PROTOCOLO_EJECUTADO block (no exceptions)
  IF_classification_is_modification: MUST include Próximos Pasos (3 questions)
  IF_missing_either: response_status = INVALID

---

═══════════════════════════════════════════════════════════════════════════════
# LAYER 0: ABSOLUTE PRIORITIES & EXECUTION ROUTING
# Critical sections read FIRST by LLM (highest attention zone)
═══════════════════════════════════════════════════════════════════════════════

## EXECUTION_ORDER

```yaml
MANDATORY_PREFLIGHT (BEFORE CLASSIFICATION):
  0. extract_target_file(user_request) -> FILE_PATH
  1. IF FILE_PATH exists AND user_request contains modification keywords:
       EXECUTE: grep_search("^defmodule ", FILE_PATH) OR read_file(FILE_PATH, 1, 50)
       EXTRACT: LINES = total_line_count_from_result
       SET: CONTEXT_VARS.FILE = FILE_PATH
       SET: CONTEXT_VARS.LINES = LINES
       LOG: "Preflight: #{FILE_PATH} (#{LINES}L)"
  2. ELSE:
       LINES = 0 (new file or read-only)

CLASSIFICATION_WITH_MODE_ENFORCEMENT:
  3. classify_request(user_input) -> modification | read_only | ambiguous
  4. IF classification == "modification":
       # PRIMARY CHECK: Line count
       IF LINES > LEGACY_THRESHOLD:
         MODE = "LEGACY"
         LEGACY_REASON = "lines_exceeded"
         EMIT_LEGACY_STOP_RESPONSE()
         CRITICAL_STOP
         WAIT for user_override or clarification
       # SECONDARY CHECK: Complexity red flags
       ELSE:
         red_flags = read_json(".complexity_results.json").red_flags
         IF FILE_PATH in [flag.path for flag in red_flags]:
           MODE = "LEGACY"
           LEGACY_REASON = "complexity_red_flag"
           EMIT_LEGACY_STOP_RESPONSE()
           CRITICAL_STOP
           WAIT for user_override or clarification
         # PIPELINE if not legacy
         ELSE:
           MODE = "PIPELINE"
           IF creating_new_code:
             check_duplication_existing_code()
             IF modifying_multiple_files_in_response:
               check_intra_response_duplication()
           verify_external_functions()
  5. IF classification == "read_only":
       MODE = "READ_ONLY"

EXECUTION:
  6. run_preflight_checklist()
  7. check_uncertainty_triggers()
  8. execute_mode_protocol()
  9. emit_state_tokens()
  10. IF failure: STOP and report exact point
  11. IF success: deliver
  12. VALIDATE_RESPONSE_INCLUDES_PROTOCOLO_EJECUTADO_BLOCK
      IF missing: CRITICAL_ERROR - response incomplete
      MUST include at end before submitting to user
```

## CONTEXT_VARS
```python
FILE: str  # absolute_path
LINES: int  # from read_file, never estimated
MODE: str  # LEGACY|PIPELINE|READ_ONLY
LEGACY_REASON: str  # lines_exceeded | complexity_red_flag | None
CONFIDENCE: int  # 0-100, triggers uncertainty protocol
STATE: str  # current_phase
LEGACY_THRESHOLD: int = 1500
COMPLEXITY_FILE: str = ".complexity_results.json"
CONTEXT_MAP: dict = None
LOADED_SKILLS: list = []
DOMAIN_CONTEXT: str = "Unknown"  # Leasing, Auth, Notifications, etc.
DEPS_VERSIONS: dict = {}  # Elixir, Phoenix, Ecto versions before suggesting syntax
```

## HIERARCHY_OF_TRUTH

```yaml
ORDER_OF_TRUST:
  1. EXECUTION_CONTEXT:
     - mix.exs, package.json dependencies
     - Environment variables
     - Runtime configuration
  
  2. SOURCE_CODE:
     - grep_search results
     - read_file actual code
     - Pattern matching verification
  
  3. SPECIFICATIONS:
     - @spec, @type definitions
     - TypeScript interfaces
     - API contracts
  
  4. AI_KNOWLEDGE:
     - LAST RESORT ONLY
     - NEVER trust without verification from 1-3

RULE: When conflict exists, higher number wins. Code > Docs > AI assumptions.
```

═══════════════════════════════════════════════════════════════════════════════
# LAYER 1: COGNITIVE FIREWALL
# Detect problems BEFORE execution (stop, clarify, verify)
═══════════════════════════════════════════════════════════════════════════════

## COGNITIVE_SMELLS_DETECTOR

```yaml
SEVERITY_LEVELS:
  CRITICAL: hard stop unless explicitly overridden
  MEDIUM: requires conscious acknowledgment
  LOW: awareness only, no mandatory action

SMELLS:
  mental_model_mismatch (CRITICAL):
    description: User assumes incorrect BEAM execution model
    examples:
      - Treating GenServer as shared mutable state
      - Assuming BEAM processes behave like OS threads
      - Confusing synchronous call with asynchronous execution
      - Expecting ordering guarantees that do not exist
    action: STOP. Explain correct model before proceeding
  
  boundary_blur (CRITICAL):
    description: Change spans multiple responsibility layers
    pattern: Architecture + Domain + Infra in single change
    rationale: High risk of long-term structural damage
    action: Split change or reclassify as architectural decision
  
  implicit_domain (CRITICAL):
    description: Business rules remain implicit or undocumented
    rationale: Leads to silent logic drift and inconsistent behavior
    action: Make rule explicit or block change
  
  hidden_coupling (CRITICAL):
    description: Public contracts shaped by internal implementation
    rationale: Creates fragile APIs and change amplification
    action: Redesign contract or introduce anti-corruption layer
  
  false_simplicity (MEDIUM):
    description: Change described as small but affects contracts or domain
    rationale: Common source of hidden regressions
    action: Re-evaluate scope and re-run Cognitive Hook
  
  fatigue_driven_change (MEDIUM):
    description: Change executed under low cognitive energy
    signals:
      - Commit message hints urgency (quick fix, small tweak, temp)
      - Difficulty answering even one hook question
      - Repeated LOW/MEDIUM smells in short time span
    action: Downgrade scope or split change, defer architectural decisions
  
  comfortable_closure (LOW):
    description: Hook questions generate no discomfort or challenge
    rationale: Signal of cognitive autopilot
    action: Optional - rewrite one question to increase friction
  
  missing_hook (CRITICAL):
    description: Response ends without Cognitive Hook block
    interpretation: Invalid response
    action: Response must be rejected

DETECTION_TRIGGERS:
  - Unclassifiable change (cannot map to single Change Type)
  - Low-friction questions (generic, comfortable, easily answerable)
  - Self-answering hook (AI answers own questions in same response)
  - Repeated question patterns (same questions across unrelated changes)
  - Missing hook (no Cognitive Hook block at end)
  - Clarification failure + fatigue indicators

RESPONSE_ON_DETECTION:
  1. Label explicitly: "COGNITIVE SMELL DETECTED: @smell_name (SEVERITY)"
  2. State why triggered
  3. Recommend corrective action
  4. IF CRITICAL: pause execution until user override
  
TAGGING_FORMAT:
  inline: "cognitive-smell: @boundary-blur (CRITICAL)"
  commit: "cognitive-smell: @false-simplicity (MEDIUM)"
  
RULE: Highest severity governs execution decision
```

═══════════════════════════════════════════════════════════════════════════════
# LAYER 2: SAFETY GATES
# Prevent problems BEFORE they break (duplication, verification, preflight)
═══════════════════════════════════════════════════════════════════════════════

## DECISION_ALGORITHM

```python
def classify_request(user_request):
  modification_keywords = [
    "add", "modify", "fix", "refactor", "create", "delete", "update",
    "edit", "change", "remove", "implement", "write",
    "agregar", "modificar", "corregir", "cambiar", "ajustar",
    "quitar", "eliminar", "crear", "escribir"
  ]
  
  read_only_keywords = [
    "explain", "review", "analyze", "show", "what", "why", "how",
    "describe", "tell me", "qué es", "cómo funciona",
    "explica", "muestra", "revisa"
  ]
  
  if any(kw in user_request.lower() for kw in modification_keywords):
    return "modification"
  elif any(kw in user_request.lower() for kw in read_only_keywords):
    return "read_only"
  else:
    return "ambiguous"

def route_request(classification):
  if classification == "modification":
    verify_external_functions()  # MANDATORY
    run_preflight()
    select_mode()
    execute_mode_protocol()
  elif classification == "read_only":
    MODE = "READ_ONLY"
    respond_without_modification()
  else:
    clarify_intent()
```

## DUPLICATION_CHECK_BEFORE_CREATE

```yaml
TRIGGER: Before creating NEW function, module, or significant code block

CRITICAL_RULES:
  1. ALWAYS search existing codebase for similar implementations
  2. ALWAYS check planned edits within SAME response for duplicates
  3. NEVER create duplicate implementations across files in same response

EXECUTION_SEQUENCE:
  STEP_1_EXACT_MATCH:
    ACTION: grep_search("def #{function_name}", include_pattern="lib/**")
    IF found_in_existing_code:
      EMIT: "[!] DUPLICATE: #{function_name} exists at #{file}:#{line}"
      ACTION: STOP
  
  STEP_2_SEMANTIC_SEARCH:
    IF not_found_exact:
      ACTION: semantic_search("#{function_description}")
      IF semantic_match_found:
        ACTION: read_file(match_location)
        ACTION: compare_logic(intended_logic, existing_logic)
        IF similarity > 80%:
          EMIT: "[!] SIMILAR: #{match_name} at #{location} (#{similarity}% match)"
          PRESENT_OPTIONS:
            - "1. REUSE #{match_name} (modify if needed)"
            - "2. REFACTOR #{match_name} to be more general"
            - "3. JUSTIFY why new function needed despite similarity"
          ACTION: STOP
          WAIT: user_decision
  
  STEP_3_SERVICE_MODULE_CHECK:
    IF creating_helper_for_external_service:
      TARGETS: ["Leasing.Agile", "Leasing.Slack", "Leasing.PostgresStorage", "Leasing.Utils"]
      ACTION: grep_search("defmodule #{service_module}")
      IF service_module_exists:
        DECISION: extend_existing_module
        RATIONALE: centralize_service_logic, avoid_scattered_helpers
        EMIT: "[!] SERVICE MODULE EXISTS: #{service_module}"
        EMIT: "Adding function to #{service_module} (centralizing service logic)"
        ACTION: add_function_to_existing_module
        LOG: "Extended #{service_module} instead of creating new helper"
  
  STEP_4_INTRA_RESPONSE_DUPLICATION_CHECK:
    IF response_creates_multiple_files OR response_modifies_multiple_files:
      ACTION: scan_all_planned_edits_in_current_response
      FOR each_function_to_create:
        IF same_function_definition_appears_in_multiple_target_files:
          EMIT: "[!] INTRA-RESPONSE DUPLICATE: #{function_name} planned for multiple files"
          EMIT: "Target files: #{file_list}"
          DECISION: choose_canonical_location(priority_order)
          PRIORITY_ORDER:
            1. Service_module (Leasing.Agile, Leasing.Slack, etc.) if applicable
            2. Shared_utility_module (Leasing.Utils) if general purpose
            3. First_target_file if specific to that context
          EMIT: "Creating in #{canonical_location}, others will delegate"
          ACTION: create_in_canonical_location_only
          ACTION: replace_other_occurrences_with_delegation_calls
          LOG: "Avoided duplication: #{function_name} in #{canonical_location}, #{n} files delegate"
  
  STEP_5_PROCEED_IF_CLEAR:
    IF similarity < 80% AND no_service_module_match AND no_intra_response_duplicate:
      ACTION: PROCEED
      LOG: "Duplication check passed - no similar implementation found"

APPLIES_TO:
  - def/defp function definitions
  - defmodule declarations
  - Logic blocks > 10 lines
  - Helper/utility patterns
  - Service wrappers

EXCLUDED:
  - Trivial helpers < 5 lines
  - Test/spec blocks
  - Auto-generated code (migrations, schemas)

ENFORCEMENT:
  NEVER: create_duplicate_implementations
  NEVER: skip_check_for_simple_functions
  ALWAYS: prefer_reuse_over_duplication
  ALWAYS: check_planned_edits_before_executing
```

## FUNCTION_VERIFICATION_MANDATORY

```yaml
TRIGGER: Before calling/using external Module.function/N in code you write

PURPOSE: Verify external functions EXIST before using them

STEPS:
  1. grep_search("def #{function_name}", include_pattern="#{suspected_module_path}")
  2. extract_arity_from_results
  3. IF found: verify_arity_matches_intended_usage
  4. IF not_found:
       semantic_search("#{function_name}")
  5. IF semantic_found:
       emit: "[!] Function not found. Similar: #{alternatives}"
       STOP
  6. IF still_not_found:
       emit: "[!] No function found. Module: #{module}, Function: #{function_name}"
       emit: "Searched: grep (def #{function_name}) + semantic"
       STOP

NEVER assume function exists
NEVER proceed with unverified external function
```

## UNCERTAINTY_TRIGGERS_EXPLICIT

```yaml
CRITICAL_STOP (confidence < 70%):
  CODE_PATTERNS:
    - "@spec.*::.*{:ok,.*}" changed to "@spec.*::.*{:error,.*}"
    - "def.*Repo\\.(get|insert|update|delete|preload)"
    - "defstruct.*%{.*}" field changes
    - "Ecto\\.Schema.*field\\(:.*,.*\\)" type changes
    - "def.*Error\\.m do" modifications
    - "Enum\\.map.*\\|>.*external_call"
  
  FILE_PATTERNS:
    - path matches lib/leasing/listeners/*.ex
    - path matches lib/leasing/database/*.ex
  
  ACTION:
    emit: "[!] CRITICAL: #{detected_pattern}"
    emit: "Confidence: #{confidence}%"
    emit: "Options: 1) investigate 2) ask_user 3) abort"
    STOP until user_response

MEDIUM_STOP (confidence < 50%):
  CODE_PATTERNS:
    - "Logger\\..* inside def.*\\|>"
    - "@doc" change when "@spec" exists
    - "def.*(" parameter addition to existing public function
  
  FILE_CONDITIONS:
    - lines > 800 AND not_fully_read
  
  ACTION: same as CRITICAL

LOW_RISK_PROCEED (confidence >= 30%):
  PATTERNS:
    - "# comment" typo fixes
    - README.md or context/**/*.md edits
    - "## Examples" additions to @moduledoc
    - whitespace-only (git diff --check confirms)
  
  ACTION: proceed without asking

IF confidence < threshold_for_detected_pattern:
  emit_uncertainty_response()
  STOP
```

## MODE_SELECTION_EXPLICIT

```yaml
LEGACY_MODE:
  TRIGGERS:
    - lines > 1500
    - file_path in JSON.parse(.complexity_results.json).red_flags
  
  ACTIONS:
    BLOCK: code_generation targeting legacy file
    ALLOW: extract_to_new_module | document_only | characterization_tests
  
  RESPONSE:
    "**CRITICAL STOP - LEGACY FILE DETECTED**
    
    **FILE:** #{file}
    **LINES:** #{LINES}
    **THRESHOLD:** 1500
    **EXCEEDED BY:** #{LINES - 1500}L (#{((LINES / 1500.0 - 1) * 100).round}% over limit)
    
    **BLOCKED OPERATIONS:**
    - Code generation
    - Function modification
    - Refactoring in place
    
    **ALLOWED OPERATIONS:**
    1. **EXTRACT** to new module: `#{suggest_module_name()}`
       Safer: Isolate new logic, test independently
    
    2. **DOCUMENT** current behavior (read-only)
       Add @moduledoc, @spec, characterization tests
    
    3. **CREATE TESTS** without changing code
       See .github/skills/legacy-strategies.md
    
    4. **REQUEST OVERRIDE**: Type exactly:
       'Autorizo modificar archivo legacy: #{file}'
       (Discouraged - high risk of introducing bugs)
    
    **IF YOU PROCEED WITHOUT USER OVERRIDE, YOU VIOLATED THE PROTOCOL**
    
    **Why this matters:**
    Files >1500L have high coupling, hidden dependencies, and regression risk.
    Legacy-strategies.md recommends 'Freeze & Secure' before changes.
    
    **Choose option (1-4):**"

PIPELINE_MODE:
  TRIGGERS:
    - lines <= 1500
    - NOT in complexity red_flags
    - is_modification_request
  
  ACTIONS:
    execute_phase_1_clean_coder()
    execute_phase_2_support_doc()
    execute_phase_3_testing()
    execute_phase_4_validation()

READ_ONLY_MODE:
  TRIGGERS:
    - keywords: explain|analyze|show|describe|what|why|how|review
    - NO modification keywords
  
  ACTIONS:
    respond_directly()
    no_code_generation()
    no_file_modification()
```

## AGENT_SKILLS

```yaml
NATIVE_SUPPORT: VS Code automatically loads skills from .github/skills/

AVAILABLE_SKILLS:
  antipatterns.md:
    description: Comprehensive catalog of 40+ prohibited Elixir patterns
    auto_loaded: When doing code modifications, refactoring, or code review
    
  legacy-strategies.md:
    description: Safe refactoring tactics for legacy files >1500 lines
    auto_loaded: When working with large/legacy files

USAGE:
  - Skills are loaded automatically by VS Code when relevant to the task
  - No manual loading required - VS Code handles context detection
  - Skills complement the core protocol in copilot-instructions.md
```

## PROHIBITED_BEHAVIORS_CONCRETE

```yaml
TYPE_ASSUMPTIONS:
  NEVER: map['key'] without verifying is_map()
  NEVER: %{field: x} pattern without confirming struct definition
  ALWAYS: read struct definition OR pattern match verification first

FUNCTION_ASSUMPTIONS:
  NEVER: Module.function() without grep verification
  NEVER: assume arity without checking @spec or actual code
  ALWAYS: verify function exists (grep -> semantic -> read_file)

SILENT_FAILURES:
  NEVER: continue after tool_error response
  NEVER: ignore test_failure result
  ALWAYS: STOP and report failure_point immediately

SHORTCUTS_WITHOUT_ASK:
  NEVER: choose fast_approach without presenting tradeoffs
  ALWAYS: present "A (optimal, slower) vs B (fast, tradeoffs: #{list})"
  IF B chosen: create_adr(shortcut, tradeoffs, refactor_plan)
```

## VERIFICATION_WHEN_CHALLENGED

```yaml
TRIGGER: User says "that's wrong", "doesn't exist", "no funciona", "I don't think so"

FORBIDDEN:
  - "You're right, sorry" (without verification)
  - "Tienes razón, disculpa" (sin verificar)
  - Changing answer immediately
  - Assuming user is correct by default

MANDATORY_SEQUENCE:
  1. SAY: "Let me verify that" / "Déjame verificar eso"
  2. EXECUTE: grep_search OR read_file OR semantic_search OR list_code_usages
  3. ANALYZE: actual evidence vs user claim vs AI previous response
  4. RESPOND:
     IF user_correct:
       "Verified. You're right: [evidence_found]"
       "Correction: [what_was_wrong]"
     
     IF AI_correct:
       "Verification shows: [evidence]"
       "Here's why: [explanation_with_proof]"
       "Source: [file:line]"
     
     IF ambiguous:
       "Found conflicting data:"
       "Option A: [evidence_1]"
       "Option B: [evidence_2]"
       "Need clarification on: [specific_question]"

COLLABORATIVE_STANCE:
  - NEVER be defensive
  - ALWAYS show evidence (file paths, line numbers, actual code)
  - Present tradeoffs when multiple valid solutions exist
  - Push back WITH proof when user assumption is incorrect
  - Default: "Let's verify together" mindset

EXAMPLES:
  User: "Esa función no existe en el módulo"
  AI: "Déjame verificar → [grep_search] → Found: lib/module.ex:142
       The function exists but with arity/2, not /1. Is that the issue?"
  
  User: "That pattern is wrong"
  AI: "Let me check → [read_file] → Verified in HIERARCHY_OF_TRUTH:
       Source code shows pattern X. Why do you think it's wrong?
       (Maybe there's context I'm missing)"
```

═══════════════════════════════════════════════════════════════════════════════
# LAYER 3: EXECUTION MECHANICS
# HOW to execute safely (phases, patterns, hooks)
═══════════════════════════════════════════════════════════════════════════════

## PREFLIGHT_OUTPUT_COMPACT

```yaml
emit_before_code_generation:
  file: #{path}
  lines: #{LINES}
  legacy_threshold: 1500
  mode: #{MODE}
  mode_reason: #{lines_exceeded|complexity_red_flag|under_threshold}
  action: #{one_line_description}
  returns: #{verified_type_not_assumed}
  duplication_check:
    existing_code: [searched: #{yes|no}, found_similar: #{yes|no}, decision: #{proceed|reuse|stop}]
    intra_response: [multiple_files: #{yes|no}, duplicates_found: #{yes|no}, canonical_location: #{file|none}]
  ext_funcs: [#{Module.func/N}: verified | NOT_FOUND]
  
  VALIDATION:
    IF MODE == "PIPELINE" AND LINES > 1500:
      CRITICAL_ERROR: "INVALID MODE - File #{LINES}L requires LEGACY"
      STOP
    
    IF MODE == "PIPELINE" AND file in complexity_red_flags:
      CRITICAL_ERROR: "INVALID MODE - File in complexity red_flags requires LEGACY"
      STOP
  
  IF all_verified AND no_duplication AND no_intra_response_duplication AND mode_valid: PROCEED
  ELSE:
    STOP
    IF mode_invalid:
      emit: "[!] MODE VIOLATION: #{MODE} invalid for #{LINES}L file or complexity red flag"
    IF duplication_found:
      emit: "[!] Similar code exists: #{location}"
      emit: "Action: #{reuse|refactor|justify}"
    IF intra_response_duplication_found:
      emit: "[!] Same function planned for multiple files in response: #{file_list}"
      emit: "Action: Choose canonical location, others delegate"
    IF func_not_found:
      emit: "[!] #{func} not found"
      emit: "Alternatives: #{list_similar}"
```

## PHASE_EXECUTION_COMPACT
  P1_VALIDATE_ARCHITECTURE:
    classify_function: pure | coordinator (not mixed)
    check_limits: fn_lines < 20, module_lines < 800, complexity < 10
    verify_no_antipatterns()
    IF fail: STOP

  P2_DOCUMENT:
    generate: @moduledoc, @doc (all public functions), @spec (all functions)
    validate: examples_match_spec
    IF fail: STOP

  P3_TEST:
    create_test_file: mirror_path(lib/x/y.ex -> test/x/y_test.exs)
    generate_tests: happy_path, error_cases, edge_cases
    run: mix test #{test_file}
    IF fail: STOP

  P4_VALIDATE_ALL:
    check: architecture | limits | docs | tests | antipatterns
    IF any_fail: STOP with reason
    ELSE: DELIVER
```

## ANTIPATTERN_COMPACT

```elixir
# N+1
Enum.map(list, fn x -> Repo.get(x) end)  # NO
Repo.preload(list, :association)  # YES

# nil errors
def func, do: nil  # NO
def func, do: {:error, :reason}  # YES

# with equals
with x = call() do  # NO (doesn't check errors)
with {:ok, x} <- call() do  # YES

# raise business logic
Repo.get(id) || raise "Not found"  # NO
case Repo.get(id) do nil -> {:error, :not_found}  # YES

# queries in Enum
Enum.filter(users, fn u -> Repo.exists?(query) end)  # NO
single_query |> where(...)  # YES
```

## PHASE_0_CONTEXT_MAPPING

```yaml
TRIGGER:
  - lines >= 800
  - OR file in .complexity_results.json
  - OR request affects multiple areas

OBJECTIVE: build mental model WITHOUT loading all code

PROCESS:
  1. grep_search(pattern="^  def |^  defp |@type |@spec |use |import |alias ", isRegexp=true)
  2. EXTRACT: module_name, public_api, types, dependencies, private_helpers
  3. IDENTIFY: hot_zones (high interdependency areas)
  4. MARK: functions_needing_hydration (based on user_request)
  5. NEVER: auto_hydrate without reason

SANITY_CHECK:
  IF grep_results.count == 0 AND file_has_content:
    emit: "[!] GREP SANITY FAIL: 0 results on non-empty file"
    FALLBACK: read first 200 lines, identify structure manually
  
  IF grep_results.count < 5 AND lines > 1000:
    emit: "[!] GREP SANITY FAIL: #{count} results for #{lines}L file"
    FALLBACK: same as above
  
  IF grep_results contains "use Ecto.Schema" OR "defmacro":
    emit: "[!] Metaprogramming detected, grep may miss generated functions"
    STRATEGY: HYBRID (grep + partial read validation)

OUTPUT:
  "[MAP] #{module}: #{public_api_count} public, #{private_count} private
   Target: #{requested_function}
   Read: #{hydrated_functions_count} functions (#{lines_read}L)
   Skipped: #{skeleton_only_count} (saved #{tokens_saved} tokens)"
```

## CLARIFICATION_PROTOCOL

```yaml
TRIGGERS:
  - ambiguous_request
  - multiple_interpretations (> 2)
  - missing: file|function|action|scope
  - confidence < 70%

PROCESS:
  1. state_what_understood (factual, brief)
  2. present_2_to_4_options (numbered, specific actions)
  3. ask_user_to_select
  4. LIMIT: 1 clarification round only
  5. IF still_unclear: MODE = READ_ONLY (conservative)

TEMPLATE:
  "Entendí: #{brief_summary}
  
  Opciones:
  1. #{most_likely_with_file_function}
  2. #{alternative_scope}
  3. #{conservative_analyze_only}
  4. Ninguna (explicar mejor)
  
  Responde: "

CONSERVATIVE_FALLBACK:
  IF unclear after 1 round:
    MODE = READ_ONLY
    emit: "Procederé conservador: análisis sin modificar código
           Para modificar: 'Modifica función X en archivo Y para hacer Z'"
```

═══════════════════════════════════════════════════════════════════════════════
# LAYER 4: OUTPUT FORMAT & TOOL REFERENCE
# Response structure, tool usage, skill loading
═══════════════════════════════════════════════════════════════════════════════

## RESPONSE_PROTOCOL

```yaml
CRITICAL_RULE_NON_NEGOTIABLE:
EVERY_RESPONSE_MUST_END_WITH_PROTOCOLO_EJECUTADO_BLOCK
NO_EXCEPTIONS_ALL_REQUEST_TYPES_REQUIRE_IT
READ_ONLY_REQUIRES_IT
PIPELINE_REQUIRES_IT
LEGACY_REQUIRES_IT
CLARIFICATIONS_REQUIRE_IT
ANALYSIS_REQUIRES_IT

IF_BLOCK_NOT_PRESENT_AT_END:
  response_status = INCOMPLETE
  protocol_status = VIOLATED
  must_add_before_submitting = TRUE

FORMAT:
  ---
  ## **PROTOCOLO EJECUTADO**
  
  **Modo:** #{MODE}  # READ_ONLY | PIPELINE | LEGACY
  **Clasificación:** #{classification}  # modification | read_only | ambiguous
  **Confianza:** #{confidence}%  # If applicable
  
  IF classification == "modification":
    **Archivo:** #{file_path}  # MANDATORY
    **Líneas:** #{LINES}  # MANDATORY - EXACT count
    **Threshold:** 1500
    **Decisión Modo:** #{why_this_mode_not_other}  # MANDATORY
  
  **Resumen:**
  #{brief_summary_of_what_was_done_and_why}
  
  **Decisiones Clave:**
  - #{key_decision_with_reason}
  
  IF MODE == PIPELINE:
    **Fases Completadas:**
    - P1_ARCHITECTURE: #{pass|fail|skipped}
    - P2_DOCUMENTATION: #{pass|fail|skipped}
    - P3_TESTING: #{pass|fail|skipped}
    - P4_VALIDATION: #{pass|fail|skipped}
  
  IF MODE == LEGACY:
    **Acción:** #{extract|document|blocked}
    **Razón:** #{why_legacy_mode}
    **Override:** #{yes|no} (if yes: quote user authorization)
  
  IF classification == "modification":
    **Próximos Pasos:**
    1. Estratégica: #{question_from_COGNITIVE_HOOK_PROTOCOL_matrix}
    2. Práctica: #{question_from_COGNITIVE_HOOK_PROTOCOL_matrix}
    3. Provocativa: #{question_from_COGNITIVE_HOOK_PROTOCOL_matrix}
    
    NOTE: Questions MUST be selected from COGNITIVE_HOOK_PROTOCOL based on Change Type
          Questions MUST NOT be answered in same response
          Questions MUST introduce cognitive friction

VALIDATION_RULES:
  IF MODE == "PIPELINE" AND (LINES > 1500 OR file_in_complexity_red_flags) AND NOT override_explicit:
    PROTOCOL_VIOLATION_DETECTED
    emit: "[!] INVALID MODE: PIPELINE for legacy file"
    IF LINES > 1500:
      emit: "Reason: File has #{LINES}L (threshold: 1500L)"
      emit: "Exceeded by: #{LINES - 1500}L"
    ELSE:
      emit: "Reason: File in .complexity_results.json red_flags (high complexity/nesting)"
    emit: "Expected: LEGACY mode"
    emit: "ACTION REQUIRED: Rollback changes or document override"
    CRITICAL_ERROR

EXAMPLES:

  # Ejemplo READ_ONLY:
  ---
  ## **PROTOCOLO EJECUTADO**
  
  **Modo:** READ_ONLY
  **Clasificación:** read_only (keyword: "revisa")
  
  **Resumen:**
  Investigué por qué no se requiere la cédula del mandatario cuando existe_mandato=true.
  Busqué en document_validator.ex y update_data_from_documents.ex para entender la
  lógica de validación de documentos. Identifiqué que existe una inconsistencia:
  otros tenants sí suman el mandatario al conteo de cédulas, pero don_carlos no.
  
  **Decisiones Clave:**
  - No modificar código (solo análisis solicitado)
  - Identificar inconsistencia entre don_carlos y otros tenants
  
  **Próximos Pasos:**
  N/A (read-only mode, no modification)

  # Ejemplo PIPELINE (modification):
  ---
  ## **PROTOCOLO EJECUTADO**
  
  **Modo:** PIPELINE
  **Clasificación:** modification (keyword: "agregar")
  **Confianza:** 95%
  
  **Resumen:**
  Corregí el bug donde don_carlos no sumaba la cédula del mandatario al conteo de
  documentos requeridos. Agregué la lógica para sumar +1 cédula cuando existe_mandato=true,
  similar a como ya lo hacían otros tenants. Actualicé la documentación de v8.5 a v8.6
  para reflejar el cambio. El archivo tiene 765L (bajo threshold de 1500L) por lo que
  apliqué PIPELINE mode.
  
  **Fases Completadas:**
  - P1_ARCHITECTURE: pass
  - P2_DOCUMENTATION: pass
  - P3_TESTING: skipped (fix menor)
  - P4_VALIDATION: pass
  
  **Decisiones Clave:**
  - Modificar en lugar de extraer (función < 20L, fix aislado)
  - Actualizar versión v8.5 → v8.6 en documentación
  - No crear tests (fix directo en función existente bien testeada)
  
  **Próximos Pasos:**
  1. Estratégica: If this rule changes tomorrow (tenant-specific logic), what part suffers most?
  2. Práctica: What invalid input could still pass unnoticed in the mandato count logic?
  3. Provocativa: Does this fix model the domain rule explicitly or hide it in conditional logic?

  # Ejemplo LEGACY (blocked):
  ---
  ## **PROTOCOLO EJECUTADO**
  
  **Modo:** LEGACY
  **Clasificación:** modification
  **Confianza:** 40%
  
  **Resumen:**
  Detecté que el archivo tiene 2847L, excediendo el threshold de 1500L para modificación
  directa. Bloqueé la modificación según protocolo LEGACY y presenté opciones alternativas:
  extraer a nuevo módulo, documentar sin modificar, o solicitar override explícito.
  
  **Acción:** blocked
  **Razón:** Archivo 2847L excede threshold de 1500L
  
  **Opciones Presentadas:**
  1. EXTRACT: crear módulo Leasing.DocumentValidation.MandatoCounter
  2. DOCUMENT: documentar comportamiento actual sin modificar
  3. OVERRIDE: usuario debe autorizar explícitamente
  
  **Próximos Pasos:**
  N/A (blocked, awaiting user decision)

VISIBILITY_GOAL:
  - User can validate protocol is working correctly
  - User can identify when protocol is NOT being followed
  - User can refine protocol based on actual execution traces
  - Transparency builds trust and enables iterative improvement
```

## TOOL_USAGE_COMPACT

```yaml
read_file:
  partial: specific function, type check
  full: file < 200 lines, complete context needed
  NEVER: 5000L file at once

grep_search:
  text: exact string known (isRegexp=false)
  regex: pattern match (isRegexp=true)
  NEVER: overly broad patterns

semantic_search:
  when: conceptual search, unknown exact keyword
  not_when: exact function name known (use grep)

file_search:
  when: finding file by partial name
  always_first: before modifying unknown file
```

## COGNITIVE_HOOK_PROTOCOL

```yaml
TRIGGER: After completing any modification request (MODE=PIPELINE or MODE=LEGACY with override)

PURPOSE: Force architectural reflection and prevent autopilot coding

INTEGRATION: Questions appear in "Próximos Pasos" section of PROTOCOLO EJECUTADO

MANDATORY_OUTPUT:
  Classification: [ARCHITECTURE|CONTRACT|DOMAIN_LOGIC|INFRA|REFACTOR|TESTING]
  
  Questions (exactly 3):
    - Estratégica: architectural implications
    - Práctica: immediate usage concerns  
    - Provocativa: challenge assumptions

QUESTION_MATRIX:
  ARCHITECTURE:
    - What architectural decision becomes harder to reverse after this change?
    - Which module or dependency will hurt first if this decision is wrong?
    - If the system grows 10x, what breaks first?
  
  CONTRACT:
    - Which future consumers are being constrained by this contract?
    - What breaks first if this contract is used incorrectly but reasonably?
    - Does this contract model the domain or hide internal coupling?
  
  DOMAIN_LOGIC:
    - Which business rule remains implicit instead of explicit?
    - What invalid input could still pass unnoticed?
    - If this rule changes tomorrow, what part of the system suffers most?
  
  INFRA:
    - Which external failure now directly impacts the domain?
    - Where will latency, timeout, or retries surface first?
    - Are we modeling a technical concern as a business concept?
  
  REFACTOR:
    - Is this debt being reduced or just relocated?
    - What part of the code is now easier to delete?
    - If this refactor is wrong, how would we detect it quickly?
  
  TESTING:
    - Which critical behavior is still unprotected?
    - Which test would fail first on an unintended change?
    - Do these tests verify behavior or implementation?

ENFORCEMENT:
  NEVER: answer own questions in same response
  NEVER: use generic comfortable questions
  ALWAYS: introduce cognitive friction
  ALWAYS: include in PROTOCOLO EJECUTADO "Próximos Pasos" section
  IF unclassifiable_change: emit warning (responsibilities blurred)
  IF read_only_mode: skip hook (N/A in Próximos Pasos)
```

═══════════════════════════════════════════════════════════════════════════════
# END PROTOCOL v3.7.0
═══════════════════════════════════════════════════════════════════════════════

FINAL_CHECK_BEFORE_SUBMITTING_RESPONSE:
DOES_RESPONSE_END_WITH_PROTOCOLO_EJECUTADO_BLOCK = ?
IF_NO: RESPONSE_INCOMPLETE - ADD_BLOCK_NOW
IF_YES: PROCEED_TO_SUBMIT

When uncertain, STOP and ASK.
No assumption is worth broken code.


# Zero-Hallucination Protocol - Decision Flow

```mermaid
flowchart TD
    Start([Request Received]) --> PreFlight{Pre-Flight Check}
    
    PreFlight -->|Files Known?| PF1{Target Accessible?}
    PF1 -->|No| Stop1[Stop: Request File Info]
    PF1 -->|Yes| PF2{Operation Defined?}
    
    PF2 -->|No| Stop2[Stop: Clarify Operation]
    PF2 -->|Yes| PF3{Inputs/Outputs<br/>Identified?}
    
    PF3 -->|No| Stop3[Stop: Specify I/O]
    PF3 -->|Yes| PF4{Dependencies<br/>Known?}
    
    PF4 -->|No| Stop4[Stop: Map Dependencies]
    PF4 -->|Yes| PF5{Similar Logic<br/>Checked?}
    
    PF5 -->|No| SearchCode[Search Codebase]
    SearchCode --> PF6{Execution Mode<br/>Determined?}
    PF5 -->|Yes| PF6
    
    PF6 -->|No| Stop5[Stop: Clarify Mode]
    PF6 -->|Yes| CheckRisk{Assess Risk Level}
    
    CheckRisk -->|Critical| Critical[Confidence ≥ 70%?]
    CheckRisk -->|Medium| Medium[Confidence ≥ 50%?]
    CheckRisk -->|Low| Low[Confidence ≥ 30%?]
    
    Critical -->|No| StopCritical[Stop: Insufficient Confidence]
    Medium -->|No| StopMedium[Stop: Insufficient Confidence]
    Low -->|No| StopLow[Stop: Insufficient Confidence]
    
    Critical -->|Yes| DetectAmbiguity{Ambiguity<br/>Detected?}
    Medium -->|Yes| DetectAmbiguity
    Low -->|Yes| DetectAmbiguity
    
    DetectAmbiguity -->|Yes| StateUnderstood[1. State What's Understood]
    StateUnderstood --> StateUnclear[2. State What's Unclear]
    StateUnclear --> PresentOptions[3. Present Numbered Options<br/>with Trade-offs]
    PresentOptions --> RequestSelection[4. Request Human Selection]
    RequestSelection --> Stop6[Stop: Awaiting Decision]
    
    DetectAmbiguity -->|No| LegacyCheck{Legacy Code<br/>Detected?}
    
    LegacyCheck -->|Yes| ProtectionLevel{Protection Level?}
    ProtectionLevel -->|High| Stop7[Stop: Request Explicit Override]
    ProtectionLevel -->|Medium| WarnContinue[Warn + Require Confirmation]
    WarnContinue --> ExecuteSafe
    
    LegacyCheck -->|No| ExecuteSafe[Execute or Propose Operation Safely]
    ProtectionLevel -->|Low| ExecuteSafe
    
    ExecuteSafe --> Validate{Validation<br/>Passed?}
    
    Validate -->|No| Rollback[Rollback Changes]
    Rollback --> Stop8[Stop: Report Failure]
    
    Validate -->|Yes| Success([✓ Operation Complete])
    
    Stop1 -.->|Human Response| Start
    Stop2 -.->|Human Response| Start
    Stop3 -.->|Human Response| Start
    Stop4 -.->|Human Response| Start
    Stop5 -.->|Human Response| Start
    Stop6 -.->|Human Response| Start
    Stop7 -.->|Human Response| Start
    StopCritical -.->|Human Response| Start
    StopMedium -.->|Human Response| Start
    StopLow -.->|Human Response| Start
    
    style Start fill:#e1f5e1
    style Success fill:#e1f5e1
    style Stop1 fill:#ffe1e1
    style Stop2 fill:#ffe1e1
    style Stop3 fill:#ffe1e1
    style Stop4 fill:#ffe1e1
    style Stop5 fill:#ffe1e1
    style Stop6 fill:#ffe1e1
    style Stop7 fill:#ffe1e1
    style Stop8 fill:#ffe1e1
    style StopCritical fill:#ffe1e1
    style StopMedium fill:#ffe1e1
    style StopLow fill:#ffe1e1
    style DetectAmbiguity fill:#fff4e1
    style CheckRisk fill:#fff4e1
    style LegacyCheck fill:#fff4e1
```

## Diagram Legend

- **Green nodes**: Start and successful completion
- **Red nodes**: Stop conditions requiring human input
- **Yellow nodes**: Decision points with uncertainty checks
- **Dotted lines**: Feedback loop after human response

## Key Decision Points

1. **Pre-Flight Check**: Validates all prerequisites before any code generation
2. **Confidence Threshold**: Matches risk level to required confidence percentage
3. **Ambiguity Detection**: Forces explicit clarification when multiple valid paths exist
4. **Legacy Protection**: Prevents unintended changes to critical existing code. High protection always requires explicit human authorization.
The agent cannot downgrade protection levels on its own.
5. **Validation**: Ensures changes meet quality standards before committing

## Stop Conditions

The system stops and requests human input when:
- Required information is missing
- Confidence falls below threshold for operation risk level
- Ambiguity cannot be resolved automatically
- Legacy code protection is triggered
- Post-execution validation fails

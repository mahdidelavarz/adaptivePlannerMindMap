# MVP Core Loop

This file converts the FigJam MVP flow into a maintainable Mermaid flowchart.

## Core loop

```mermaid
flowchart LR
  A["User opens app"] --> B{"Has unresolved tasks?"}

  B -->|Yes| C["Reconcile screen"]
  B -->|No| D["Today page"]

  C --> E{"User action"}
  E -->|Done| F["Mark completed"]
  E -->|Carry| G["Carry forward"]
  E -->|Drop| H["Drop task"]

  G --> I{"Keep same size?"}
  I -->|Yes| D
  I -->|No| J["Shrink / clarify task"]
  J --> D

  F --> K["Append event"]
  H --> K
  D --> L["User plans or works"]
  L --> K

  K --> M["Daily rollup"]
  M --> N["Weekly evaluation"]
  N --> O["AI proposes next plan draft"]
  O --> P{"User approves?"}
  P -->|Edit| Q["User edits"]
  P -->|Approve| R["Next 7-day plan"]
  Q --> R
```

## Product meaning

The first product bet is not "AI creates a perfect plan".

The first product bet is:

> If users can reconcile unfinished work without shame, they are more likely to return after slippage instead of abandoning the planner.

## Locked behavior

- No red overdue guilt wall.
- No streak-first design.
- No automatic AI plan application.
- AI drafts only.
- User approves changes.
- Append-only event log.

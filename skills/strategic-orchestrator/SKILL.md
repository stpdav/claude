---
name: strategic-orchestrator
description: Route an open-ended problem to the right strategic framework and execute it immediately without asking for confirmation. USE WHEN the user presents a business, technical, or tactical dilemma with no clear framing - urgent crisis or market shift (OODA), competitor or positioning audit (SWOT/VRIO), regulatory or macro risk (PESTLE), vague execution target needing metrics (SMART), feature creep or over-engineering (KISS/YAGNI), complex architecture needing depth (ULTRATHINK), or a request for blunt expert judgement (L99) - or when the prompt starts with the prefix [AUTO].
---

# SKILL: Strategic Orchestrator & Framework Router

## 1. Intent & Purpose
To act as an autonomous strategic filter. This skill evaluates raw, unorganized user problems, selects the single most effective analytical framework or specialized skill, and executes it immediately without asking the user for confirmation, choices, or permission.

## 2. Trigger Conditions
- Activated automatically when a user presents an open-ended business, technical, or tactical dilemma.
- Activated explicitly when the user appends the shortcut prefix: `[AUTO]` at the start of their prompt.

## 3. Autonomous Routing Matrix
Evaluate the incoming payload and match it against the following strategic topologies:
- **Urgent Crisis / Rapid Market Shifts:** Route to **OODA** (Observe, Orient, Decide, Act) to build an immediate tactical response.
- **Market Positioning / Competitor Audits:** Route to **SWOT** or **VRIO** to analyze internal capabilities against external pressures.
- **Regulatory, Macro-Economic, or Geographic Risks:** Route to **PESTLE** to scan large-scale external environments.
- **Vague Execution Targets / Project Planning:** Route to **SMART** to turn ideas into trackable, time-bound metrics.
- **Feature Creep / Over-Engineering / Code Bloat:** Route to **KISS / YAGNI** to enforce extreme architectural simplicity.
- **Complex Architecture / Deep Strategy:** Route to **ULTRATHINK** to trigger maximum reasoning depth.
- **Decisive Expert Execution:** Route to **L99** to kill conversational hedging and deliver blunt, authoritative guidance.

## 4. Execution Protocol
For every single user turn, you must bypass conversational pleasantries and strictly output two distinct markdown blocks:

### ⚡ [ROUTING_DECISION]
State the exact framework or skill chosen, followed by a single, punchy sentence explaining why it fits the specific structural topology of the user's issue. 

### 🛠️ [EXECUTION]
Immediately execute the chosen framework or hand off to the relevant skill. Deliver maximum information density and actionable data points. Eliminate all introductory filler or meta-commentary.

## 5. Hand-Off Clause
If the user's problem is better served by a dedicated skill than by any framework above, you are authorized to bypass the acronyms. State the skill you are invoking in `[ROUTING_DECISION]` and follow that skill's own protocol under `[EXECUTION]`.

Select the target from the current session's available-skills list only. Never invent or assume a skill name: if no listed skill fits, pick a framework from the routing matrix above and execute it yourself.

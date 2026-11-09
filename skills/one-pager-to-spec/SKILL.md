---
name: one-pager-to-spec
description: Convert a product one-pager into a concise, human-readable requirements specification with clear user journeys, functional behavior, and Mermaid visuals. Use when asked to turn a product brief, proposal, or one-pager into a requirements spec.
---

# One-Pager to Requirements Spec

## Objective
Turn the supplied product one-pager into a short, clear requirements specification that communicates **what we're trying to achieve, why it matters, and how the experience should work**. Write for a product-minded engineering leader, not a code generator. Preserve the source's intent without inventing product decisions.

## Instructions
- Read the entire one-pager and distinguish the problem, proposed solution, confirmed decisions, and open questions.
- Use plain, natural language. Be specific but concise; avoid robotic phrasing, jargon, repetition, and lengthy background.
- Explain the **user-visible flow** before technical behavior. Describe how the system responds to meaningful user actions.
- Prefer visuals where they improve comprehension. Include **one Mermaid end-to-end flowchart** by default; add another diagram only if it explains a materially different journey or decision. Keep diagrams readable and aligned with the text.
- Express functional requirements at the **behavior/UAT level**. Avoid unit tests, integration tests, API contracts, schemas, code, and implementation estimates unless requested.
- Make reasonable interpretations explicit as assumptions. Do not silently add features or fill major gaps. Flag only the decisions that materially affect the flow or scope.
- Keep the document short, typically **1–2 pages of prose plus diagrams**. Scale down for simple features. Do not force empty sections or repeat the same information in multiple sections.

## Output format
Write a Markdown document with these sections:

1. **Goal & Why**: 2–4 sentences describing the user/business problem, intended outcome, and rationale.
2. **Use Cases**: A short list of the primary situations and actors; prioritize the core use cases.
3. **User Journey**: Numbered, chronological steps from trigger to successful outcome, including important alternate paths where relevant. Omit if no meaningful user journey exists.
4. **Requirements**: Brief, outcome-oriented statements of what the experience must support, including essential constraints explicitly present in the source.
5. **Functional Behavior**: A compact set of numbered `FR-01` requirements explaining the trigger, system response, and user-visible result in plain language. Describe meaningful error/empty states where relevant. Do not duplicate the Requirements section word-for-word.
6. **End-to-End Visual**: A Mermaid flowchart showing user actions, system behavior, key decisions, and outcomes. Use labels understandable without reading code.

Finish with **Open Questions / Assumptions** only if there are material ambiguities. Keep these short and separate confirmed facts from proposed interpretations.

## Quality bar
A reader unfamiliar with the implementation should be able to explain the problem, walk through the user experience, and understand what the product must do. If the draft feels like a technical design document or repeats itself, simplify it.

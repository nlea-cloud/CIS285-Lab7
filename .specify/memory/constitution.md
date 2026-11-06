<!--
Sync Impact Report
Version change: unversioned scaffold -> 1.0.0
Modified principles: template placeholders -> I. Modular Layered Architecture;
  II. Separation of Concerns; III. Object-Oriented Design;
  IV. UML Class Diagrams; V. Activity and Workflow Diagrams
Added sections: Architecture and Modeling Standards; Design and Review Workflow
Removed sections: none
Follow-up TODO: Confirm the original ratification date; it is not present in the
  prior constitution scaffold or available project context.
-->

# CIS 285 Lab 7 Constitution

## Core Principles

### I. Modular Layered Architecture
The system MUST be organized into cohesive modules and explicit architectural
layers with documented responsibilities. Dependencies MUST flow through defined
interfaces in an intentional direction; circular dependencies and bypassing a
layer's public boundary are prohibited unless a documented constraint requires
an exception. Modules and layers MUST remain independently understandable and
testable. This limits change propagation and makes system behavior easier to
verify.

### II. Separation of Concerns
Each module, class, and operation MUST have a clearly identifiable
responsibility. User interaction, domain rules, persistence, and external
integration MUST be separated behind explicit contracts. Business rules MUST
not be duplicated across layers; shared rules belong in the domain layer or
another explicitly designated owner. This keeps changes localized and avoids
conflicting behavior.

### III. Object-Oriented Design
Objects MUST encapsulate state and behavior that belong together, expose only
the operations needed by their collaborators, and preserve their invariants.
Classes MUST have focused responsibilities and explicit collaboration
contracts. Composition MUST be preferred for assembling behavior; inheritance
MUST be used only when the subtype relationship preserves the parent contract.
Abstractions MUST clarify a real boundary or variation rather than add
indirection without a design need.

### IV. UML Class Diagrams
Changes that introduce or materially alter domain classes, responsibilities,
or relationships MUST include an updated UML class diagram. Diagrams MUST show
the relevant classes, key attributes and operations, and relationships with
meaningful multiplicities or generalization where applicable. The diagram MUST
be kept in version-controlled, editable form and remain consistent with the
implemented design. This makes structural decisions reviewable and exposes
unintended coupling early.

### V. Activity and Workflow Diagrams
Changes to multi-step behavior, branching processes, or workflows involving
multiple actors or system boundaries MUST include an updated UML activity or
workflow diagram. Diagrams MUST represent the relevant actors or swimlanes,
decision points, sequence, and meaningful alternate or failure paths. They
MUST remain consistent with the implemented behavior so reviewers can trace
the process from trigger to outcome.

## Architecture and Modeling Standards

Design documents and diagrams MUST identify their scope and use consistent,
readable notation. Diagram sources MUST be stored with the project documentation
or the feature they describe; rendered images MAY supplement, but MUST NOT
replace, editable sources. Where a design exception to these principles is
necessary, its rationale and affected boundaries MUST be documented for review.

## Design and Review Workflow

Before implementing a structural or workflow change, contributors MUST identify
the affected modules, layers, responsibilities, and contracts. They MUST update
applicable diagrams as part of the change and verify diagram-to-code
consistency during review. Reviews MUST check dependency direction, ownership
of responsibilities and business rules, object/class responsibilities, and
whether required diagrams cover the changed behavior. Reviewers MUST request
documented justification for exceptions; diagram updates do not replace
appropriate implementation tests.

## Governance

This constitution governs architectural and modeling decisions for the
project. Amendments MUST be made in this file and reviewed with a rationale
that describes the affected principles and any compatibility or migration
impact. Changes that add a principle or materially expand governance increment
the MINOR version; backward-incompatible removals or redefinitions increment
the MAJOR version; clarifications and non-semantic wording changes increment
the PATCH version. The version and amendment date MUST be updated with every
approved amendment.

Contributors and reviewers MUST assess relevant changes for compliance with
these principles. Exceptions MUST identify the principle, reason, scope, and
mitigation, and MUST be reviewed alongside the change. Compliance is evaluated
through design documentation, diagrams, implementation, and tests, not by
diagram presence alone.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): original adoption date unknown; confirm with project owner | **Last Amended**: 2026-11-05

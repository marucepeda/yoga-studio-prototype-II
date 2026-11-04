# AGENTS.md — Yoga Studio

## 1. Project Overview

Yoga Studio is a lightweight web application designed to support the daily operations of a small yoga studio with two instructors.

The project aims to replace manual paper-based class package tracking and booking management with a simple, reliable and transparent digital system.

The application must prioritize usability, clarity and operational efficiency over unnecessary complexity.

## 2. Product Goals

The application should help the studio:

- Manage class schedules and reservations.
- Track student class packages and available credits.
- Control class capacity.
- Organize waiting lists.
- Register attendance and cancellations.
- Maintain a reliable history of credit movements.

The experience must be intuitive for both the studio administrator and students.

## 3. Development Principles

- Follow Spec Driven Development (SDD).
- Read the relevant specifications before implementing any feature.
- Treat acceptance criteria as requirements, not suggestions.
- Prefer simple, maintainable solutions over unnecessary abstractions.
- Avoid introducing dependencies without a clear technical reason.
- Preserve existing working functionality unless a change is explicitly required.
- Do not invent business rules that have not been defined.
- Keep the code readable and organized.

## 4. Business Rules

The following rules are fundamental to the product:

- Default class capacity is 12 students.
- Class capacity must be configurable.
- A confirmed reservation requires an available credit.
- A confirmed reservation temporarily reserves one credit.
- A waiting-list position does not consume a credit.
- Waiting-list order must follow FIFO (First In, First Out).
- When a confirmed reservation is cancelled under valid cancellation conditions, the available place and credit must be released.
- Attendance must consume the corresponding credit only once.
- Credit movements must be recorded in an auditable history.
- Credit balances must not be modified directly without recording the corresponding movement.
- Duplicate reservations must be prevented.
- Class capacity must never be exceeded.

Late cancellation and absence policies must remain configurable. Do not assume or invent the studio's commercial policies.

## 5. Scope Control

The initial implementation focuses on the core class booking management feature.

The following functionalities are outside the initial scope:

- Online payments.
- E-commerce or product sales.
- Video lessons or workout routines.
- Marketing campaigns.
- Internal chat.
- Native mobile applications.
- Complex third-party integrations.
- Automated external messaging.

Do not expand the scope without explicit authorization.

## 6. Design Guidelines

The application follows a warm, minimalist and zen-inspired visual identity.

Design principles:

- Calm and uncluttered interfaces.
- Clear information hierarchy.
- Generous whitespace.
- Natural and neutral colors.
- Readable typography.
- Consistent spacing and visual components.
- Responsive layouts for desktop and mobile.

Reference color palette:

- Background: #F7F6F2
- Secondary background: #E8E1D5
- Warm neutral: #D6CEC0
- Soft green: #A5AD9B
- Muted green: #727B68
- Primary text: #343632
- White: #FFFFFF

Avoid flashy gradients, excessive decoration, gamification and conventional gym aesthetics.

Follow the `yoga-brand-design` skill whenever creating or modifying user interface elements.

## 7. Spec Driven Development Workflow

Before implementing a feature:

1. Read this file.
2. Read the corresponding feature specification.
3. Identify the acceptance criteria.
4. Review the existing project structure and implementation.
5. Identify ambiguities or contradictions before coding.

During implementation:

1. Follow the approved specification.
2. Implement only the defined scope.
3. Keep business logic separate from presentation whenever practical.
4. Avoid duplicating business rules.
5. Preserve existing functionality.

After implementation:

1. Verify every acceptance criterion.
2. Report which criteria passed or failed.
3. Provide reproducible testing steps.
4. Identify limitations or incomplete functionality.

If an acceptance criterion cannot be met, review and correct the specification before attempting to solve the issue through prompt changes alone.

## 8. Data Integrity and Security

- Do not use real student personal information during development.
- Use fictional data for examples and testing.
- Never expose credentials, API keys or secrets in source code.
- Do not commit environment variables or sensitive configuration files.
- Validate operations that modify reservations, attendance or credits.
- Prevent duplicate or inconsistent credit movements.

## 9. Communication and Reporting

When working on this project:

- Explain important technical decisions.
- Identify assumptions explicitly.
- Ask for clarification when business rules are ambiguous.
- Do not claim that a feature works unless it has been tested.
- Report limitations honestly.
- Prefer small, verifiable implementation steps.

The objective is a reliable and understandable product, not the largest possible codebase.
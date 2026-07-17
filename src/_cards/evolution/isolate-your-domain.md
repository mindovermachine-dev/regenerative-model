---
title:   Isolate Your Domain
caption: Don't let database frameworks dictate your business logic
level:   30
area:    evolution
header:
  teaser: /assets/images/cards/codememo.png
assessment:
  - id: isolate_domain_1
    question: To what degree do we keep domain decisions close to the business problem they serve?
    focus: Domain ownership
    praise: You are making the system easier to reason about.
    remedy: Group one feature around its own domain concept and remove cross-cutting shortcuts.
  - id: isolate_domain_2
    question: To what degree do we prevent technical concerns from leaking into the heart of our models?
    focus: Model purity
    praise: You are protecting the core from accidental complexity.
    remedy: Move one infrastructure-specific concern out of the domain layer and place it at the boundary.
---

As a beginner, you start noticing the situational friction caused by tightly coupled frameworks. Prevent external library constraints from polluting your core business rules. Ensure that database models, serialization schemas, and network controllers do not leak deep into your core application logic.

By isolating your business domain from these volatile framework details, you keep the heart of your application flexible, robust, and painless to upgrade over time.

**Discussion:** When you couple your business core directly to database ORM entities, upgrades to the database framework can break your entire core. Decoupling ensures your software can evolve gracefully without breaking.

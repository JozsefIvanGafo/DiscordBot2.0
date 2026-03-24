# Use of Hybrid Communication (REST + Async Events)

## Context and Problem Statement

The system is composed of multiple microservices (gateway, music, moderation, etc.) that must communicate efficiently while maintaining scalability and fault tolerance.

The problem is deciding how services should communicate: synchronously (REST), asynchronously (events), or a combination of both.

## Considered Options

* REST-only communication  
* Event-driven architecture  
* Hybrid (REST + async events)  

## Decision Outcome

Chosen option: "Hybrid communication (REST + async events)", because it balances simplicity and scalability.

- REST will be used for user-triggered commands (play, pause, skip).
- Async events will be used for state changes (track finished, queue updated).

### Consequences

* Good, because user actions get immediate feedback.
* Good, because services are decoupled via events.
* Bad, because system complexity increases.
* Bad, because requires message broker in future.
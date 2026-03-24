# Use of Component-Based and Service-Level Testing Strategy

## Context and Problem Statement

The Discord bot system is composed of multiple microservices such as gateway, music, moderation, and user services. Each component has its own responsibilities and interacts with other services through APIs or messaging.

The problem is determining how to ensure reliability, correctness, and maintainability of the system while avoiding tightly coupled tests that are hard to maintain in a distributed architecture.

## Considered Options

* End-to-end testing only  
* Unit testing only  
* Component-based and service-level testing strategy  

## Decision Outcome

Chosen option: "Component-based and service-level testing strategy", because it allows testing each service independently while also validating interactions between components.

### Consequences

* Good, because each service can be tested in isolation.
* Good, because faster feedback during development.
* Good, because failures can be traced to specific components.
* Bad, because requires test infrastructure per service.
* Bad, because integration testing between services adds complexity.
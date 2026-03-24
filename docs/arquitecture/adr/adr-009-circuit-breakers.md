# Use of Circuit Breakers for External Services

## Context and Problem Statement

The bot may rely on external services (e.g., music sources, APIs), which can fail or become slow.

The problem is ensuring system resilience when dependencies fail.

## Considered Options

* No fault tolerance  
* Retry mechanisms only  
* Circuit breaker pattern  

## Decision Outcome

Chosen option: "Circuit breaker pattern", to prevent cascading failures and improve system stability.

### Consequences

* Good, because failures are isolated quickly.
* Good, because system stability improves under load.
* Bad, because implementation adds complexity.
* Bad, because incorrect thresholds can affect availability.
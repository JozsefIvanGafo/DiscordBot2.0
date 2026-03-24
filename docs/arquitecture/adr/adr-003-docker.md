# Use of Docker for Deployment and Environment Consistency

## Context and Problem Statement

The application must be deployable across different environments (development, staging, production) with minimal configuration issues.

The problem is ensuring consistency across environments and simplifying deployment.

## Considered Options

* Manual environment setup  
* Virtual machines  
* Docker containers  

## Decision Outcome

Chosen option: "Docker containers", because they provide consistent environments and simplify deployment and scaling.

### Consequences

* Good, because environment consistency is guaranteed.
* Good, because services can be deployed independently.
* Bad, because container orchestration adds complexity.
* Bad, because debugging containerized apps can be harder.
# Use of Microservice-Oriented Architecture for Discord Bot

## Context and Problem Statement

The Discord bot will include multiple domains such as music playback, moderation, user management, and scheduled events. These features may evolve independently and require different scaling strategies.

The problem is determining how to structure the system to support scalability and maintainability.

## Considered Options

* Monolithic architecture  
* Modular monolith  
* Microservice-oriented architecture  

## Decision Outcome

Chosen option: "Microservice-oriented architecture", because it allows independent development and scaling of features such as music streaming and moderation.

### Consequences

* Good, because services can evolve independently.
* Good, because failures in one service do not affect others.
* Bad, because system complexity increases.
* Bad, because inter-service communication must be managed.
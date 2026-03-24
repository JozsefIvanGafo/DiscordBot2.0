# Inclusion of Moderation and Auto-Role Features

## Context and Problem Statement

The bot should provide additional value beyond music, including moderation tools and automatic role assignment for users.

The problem is deciding whether to include these features within the same system.

## Considered Options

* Music-only bot  
* Separate bots per feature  
* Unified bot with modular features  

## Decision Outcome

Chosen option: "Unified bot with modular features", including moderation and auto-role capabilities.

### Consequences

* Good, because it reduces the need for multiple bots.
* Good, because features can share infrastructure.
* Bad, because system complexity increases.
* Bad, because failures in shared components may affect multiple features.
# Role-Based Access Control for Bot Features

## Context and Problem Statement

The bot must allow general users to interact with music features while restricting configuration and sensitive operations to administrators.

The problem is defining a secure and flexible permission system.

## Considered Options

* No access control  
* Role-based access control  
* Permission per command  

## Decision Outcome

Chosen option: "Role-based access control", where:
- All users can control music playback  
- Only admins can configure bot settings  

### Consequences

* Good, because it prevents misuse of administrative features.
* Good, because it aligns with Discord’s role system.
* Bad, because role validation logic must be implemented carefully.
* Bad, because misconfiguration can restrict valid users.
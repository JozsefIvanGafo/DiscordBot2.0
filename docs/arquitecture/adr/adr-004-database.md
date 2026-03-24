# Use of Database for Persistent Bot Data

## Context and Problem Statement

The bot needs to store persistent data such as user birthdays, playlists, guild configurations, and moderation settings.

The problem is deciding how to manage and store this data reliably.

## Considered Options

* In-memory storage  
* File-based storage (JSON/YAML)  
* Database (SQL or NoSQL)  

## Decision Outcome

Chosen option: "Database", because it provides reliability, scalability, and structured querying.

### Consequences

* Good, because data persists across restarts.
* Good, because complex queries (e.g., scheduled birthday checks) are supported.
* Bad, because it introduces infrastructure overhead.
* Bad, because schema design must be maintained.
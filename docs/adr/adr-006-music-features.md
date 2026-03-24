# Design of Music Controller Capabilities

## Context and Problem Statement

The music module must support essential playback features while remaining responsive and stable under concurrent usage.

The problem is defining the scope of functionality for the music controller.

## Considered Options

* Minimal controls (play/stop only)  
* Standard controls (play, pause, skip)  
* Full-featured controller  

## Decision Outcome

Chosen option: "Full-featured controller", including:
- Play music  
- Pause/resume  
- Skip track  
- Add/remove from queue  
- Volume control  
- Loop (single track / queue)  
- Leave voice channel  

### Consequences

* Good, because it provides a complete user experience.
* Good, because it aligns with expectations from music apps.
* Bad, because implementation complexity increases.
* Bad, because concurrency and queue management must be handled carefully.
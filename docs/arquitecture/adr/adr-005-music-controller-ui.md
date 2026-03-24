# Use of Reaction-Based Music Controller Instead of Command-Based Interaction

## Context and Problem Statement

Traditional Discord bots rely on text commands, which can be unintuitive and clutter chat channels. A more interactive UI is desired for controlling music playback.

The problem is determining how users should interact with the music system.

## Considered Options

* Text-based commands  
* Slash commands  
* Reaction/button-based controller  

## Decision Outcome

Chosen option: "Reaction/button-based controller", because it provides a more user-friendly and interactive experience similar to a media player.

### Consequences

* Good, because user experience is improved.
* Good, because interactions are centralized in a single message.
* Bad, because state management becomes more complex.
* Bad, because handling reactions/events requires careful async handling.
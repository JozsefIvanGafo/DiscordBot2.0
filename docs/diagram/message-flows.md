# Message Flows

## Overview
This document describes the message and execution flows between the various microservices in the Discord bot architecture. The system relies on the **Gateway** service to handle all direct interactions with Discord (slash commands, events, and UI components). The Gateway delegates business logic to specialized backend services (**Music-service**, **Moderation-service**, **User-service**, **Scheduler-service**, and **Config-service**) via REST API calls and asynchronous event messaging.

---

### Music Flows

## Play Song Flow
1. User invokes the `/play <query>` slash command in a Discord channel.
2. Gateway receives the interaction event and defers the response.
3. Gateway sends a REST POST request to the Music-service containing the query, guild ID, user ID, and voice channel ID.
4. Music-service resolves the track query, connects to the specified voice channel, and adds the track to the guild's queue.
5. Music-service responds to the Gateway's REST request with the track details.
6. Gateway edits the deferred interaction with a "Track Added" or "Now Playing" UI embed.

```mermaid
sequenceDiagram
    participant U as User
    participant G as Gateway
    participant M as Music-service
    U->>G: /play <query>
    G-->>U: Defer Interaction
    G->>M: POST /api/music/play (query, guild_id, ...)
    M->>M: Connect to VC & Add to Queue
    M-->>G: 200 OK (Track Details)
    G->>U: Edit Interaction (Now Playing / Added to Queue)
```

## Pause / Resume Flow
1. User clicks the "Pause" or "Resume" button on a music UI embed, or uses the corresponding slash command.
2. Gateway receives the interaction and sends a REST PUT request to the Music-service containing the target playback state.
3. Music-service updates the audio player's state (pausing or resuming execution).
4. Music-service publishes an async `player_state_changed` event.
5. Gateway consumes the event and updates the initial UI embed to reflect the current paused/playing state.

```mermaid
sequenceDiagram
    participant U as User
    participant G as Gateway
    participant M as Music-service
    participant MB as Message Broker
    U->>G: Click Pause/Resume
    G->>M: PUT /api/music/state (paused: true/false)
    M-->>G: 200 OK
    M->>MB: Publish `player_state_changed`
    MB-->>G: Consume Event
    G->>U: Update UI Embed
```

## Skip Track Flow
1. User invokes the `/skip` command or clicks the "Skip" UI button.
2. Gateway sends a REST POST request to the Music-service to skip the current track.
3. Music-service halts current playback and shifts the queue to the next track.
4. Music-service publishes a `track_skipped` async event.
5. Gateway acknowledges the user's interaction and waits for the subsequent `track_started` event to update the UI.

```mermaid
sequenceDiagram
    participant U as User
    participant G as Gateway
    participant M as Music-service
    participant MB as Message Broker
    U->>G: /skip or Click Skip
    G->>M: POST /api/music/skip
    M->>M: Shift Queue
    M-->>G: 200 OK
    G-->>U: Acknowledge Interaction
    M->>MB: Publish `track_skipped` / `track_started`
    MB-->>G: Consume `track_started`
    G->>U: Send "Now Playing" Embed
```

## Add to Queue Flow
1. User invokes the `/play <query>` command while a song is already playing.
2. Gateway sends a REST POST request to the Music-service.
3. Music-service resolves the track, appends it to the existing queue, and returns the queue position via the REST response.
4. Gateway replies to the user with an "Added to Queue" confirmation message.

```mermaid
sequenceDiagram
    participant U as User
    participant G as Gateway
    participant M as Music-service
    U->>G: /play <query>
    G->>M: POST /api/music/play
    M->>M: Append to Queue
    M-->>G: 200 OK (Position in Queue)
    G->>U: Reply "Added to Queue (Position X)"
```

## Volume Change Flow
1. User invokes the `/volume <level>` command.
2. Gateway validates the input (e.g., 1-100) and sends a REST PUT request to the Music-service.
3. Music-service applies the volume scaling to the guild's audio player.
4. Music-service replies with a success status.
5. Gateway sends a confirmation message to the user.

```mermaid
sequenceDiagram
    participant U as User
    participant G as Gateway
    participant M as Music-service
    U->>G: /volume <level>
    G->>M: PUT /api/music/volume (level)
    M->>M: Apply Volume Scaling
    M-->>G: 200 OK
    G->>U: Confirm Volume Change
```

## Loop Mode Flow
1. User invokes the `/loop <mode>` command (modes: track, queue, off).
2. Gateway sends a REST PUT request to the Music-service specifying the loop mode.
3. Music-service updates the queue configuration for the specific guild.
4. Music-service publishes a `queue_config_updated` async event.
5. Gateway consumes the event and updates the active music UI embed to show the active loop state.

```mermaid
sequenceDiagram
    participant U as User
    participant G as Gateway
    participant M as Music-service
    participant MB as Message Broker
    U->>G: /loop <mode>
    G->>M: PUT /api/music/loop (mode)
    M->>M: Update Config
    M-->>G: 200 OK
    M->>MB: Publish `queue_config_updated`
    MB-->>G: Consume Event
    G->>U: Update UI Embed (Loop Icon)
```

## Leave Voice Channel Flow
1. User invokes the `/leave` command, or the voice channel becomes empty.
2. Gateway sends a REST DELETE request to the Music-service to destroy the guild's player.
3. Music-service clears the queue, disconnects from the voice channel, and frees resources.
4. Music-service publishes a `player_destroyed` async event.
5. Gateway consumes the event, deletes the active music UI embeds, and confirms the departure in the text channel.

```mermaid
sequenceDiagram
    participant U as User
    participant G as Gateway
    participant M as Music-service
    participant MB as Message Broker
    U->>G: /leave
    G->>M: DELETE /api/music/player
    M->>M: Disconnect & Clear
    M-->>G: 200 OK
    M->>MB: Publish `player_destroyed`
    MB-->>G: Consume Event
    G->>U: Delete UI & Confirm Departure
```

## Track Finished Flow
1. Music-service finishes playing the current audio track.
2. Music-service automatically dequeues the next track and publishes a `track_finished` async event.
3. If a new track starts, Music-service publishes a `track_started` async event.
4. Gateway consumes the `track_started` event and posts a new "Now Playing" UI embed in the designated music text channel.

```mermaid
sequenceDiagram
    participant M as Music-service
    participant MB as Message Broker
    participant G as Gateway
    participant D as Discord Channel
    M->>M: Audio Track Finishes
    M->>MB: Publish `track_finished`
    M->>M: Dequeue Next
    M->>MB: Publish `track_started`
    MB-->>G: Consume Event
    G->>D: Post "Now Playing" Embed
```

---

### Moderation Flows

## Kick User Flow
1. Moderator invokes the `/kick <user> <reason>` command.
2. Gateway sends a REST POST request to the Moderation-service with the target user ID, moderator ID, guild ID, and reason.
3. Moderation-service logs the infraction in the database.
4. Moderation-service makes an API call to Discord (either directly or via the Gateway) to execute the kick.
5. Moderation-service returns a success response to the Gateway.
6. Gateway replies to the interaction with a successful kick confirmation.

```mermaid
sequenceDiagram
    participant Mod as Moderator
    participant G as Gateway
    participant MS as Moderation-service
    participant D as Discord API
    Mod->>G: /kick <user> <reason>
    G->>MS: POST /api/moderation/kick
    MS->>MS: Log Infraction
    MS->>D: Execute Kick
    D-->>MS: 204 No Content
    MS-->>G: 200 OK
    G->>Mod: Confirm Kick
```

## Ban User Flow
1. Moderator invokes the `/ban <user> <reason>` command.
2. Gateway sends a REST POST request to the Moderation-service.
3. Moderation-service validates permissions against the Config-service (via internal REST) and logs the ban.
4. Moderation-service executes the ban action.
5. Moderation-service publishes a `user_banned` async event for audit logging.
6. Gateway replies to the moderator with a success message.

```mermaid
sequenceDiagram
    participant Mod as Moderator
    participant G as Gateway
    participant MS as Moderation-service
    participant CS as Config-service
    participant MB as Message Broker
    participant D as Discord API
    Mod->>G: /ban <user> <reason>
    G->>MS: POST /api/moderation/ban
    MS->>CS: GET /api/config/permissions
    CS-->>MS: Permissions Valid
    MS->>MS: Log Infraction
    MS->>D: Execute Ban
    MS-->>G: 200 OK
    MS->>MB: Publish `user_banned`
    G->>Mod: Confirm Ban
```

## Mute User Flow
1. Moderator invokes the `/mute <user> <duration> <reason>` command.
2. Gateway sends a REST POST request to the Moderation-service.
3. Moderation-service calculates the mute expiration time, logs the infraction, and applies the timeout/mute role.
4. Moderation-service sends a payload to the Scheduler-service to register an `expire_mute` job for the future.
5. Gateway receives the success response and confirms the mute to the moderator.

```mermaid
sequenceDiagram
    participant Mod as Moderator
    participant G as Gateway
    participant MS as Moderation-service
    participant SS as Scheduler-service
    Mod->>G: /mute <user> <duration>
    G->>MS: POST /api/moderation/mute
    MS->>MS: Log Infraction & Apply Timeout
    MS->>SS: POST /api/scheduler/jobs (expire_mute)
    SS-->>MS: 201 Created
    MS-->>G: 200 OK
    G->>Mod: Confirm Mute
```

---

### User / Config Flows

## Auto-Role Flow
1. Gateway detects an `on_member_join` Discord event.
2. Gateway publishes an async `member_joined` event.
3. Moderation-service consumes the event and makes a REST GET request to the Config-service to retrieve the guild's auto-role settings.
4. If an auto-role is configured, Moderation-service executes the role assignment.
5. Moderation-service publishes a `role_assigned` event for audit/logging purposes.

```mermaid
sequenceDiagram
    participant D as Discord
    participant G as Gateway
    participant MB as Message Broker
    participant MS as Moderation-service
    participant CS as Config-service
    D->>G: Event: `on_member_join`
    G->>MB: Publish `member_joined`
    MB-->>MS: Consume Event
    MS->>CS: GET /api/config/autorole (guild_id)
    CS-->>MS: Config (role_id)
    MS->>D: API: Add Role
    MS->>MB: Publish `role_assigned`
```

## Fetch Guild Config Flow
1. Gateway receives any command execution.
2. Gateway attempts to read the guild configuration (e.g., prefix, language) from its local cache.
3. If a cache miss occurs, Gateway sends a REST GET request to the Config-service.
4. Config-service retrieves the settings from the database (or its own Redis cache) and returns them.
5. Gateway updates its local cache and proceeds with processing the command.

```mermaid
sequenceDiagram
    participant U as User
    participant G as Gateway
    participant CS as Config-service
    U->>G: Interaction / Command
    G->>G: Check Local Cache (Miss)
    G->>CS: GET /api/config (guild_id)
    CS->>CS: DB/Redis Lookup
    CS-->>G: 200 OK (Settings Payload)
    G->>G: Update Local Cache & Process Command
```

---

### Scheduler Flows

## Birthday Notification Flow
1. Scheduler-service hits a daily cron trigger (e.g., 00:00 UTC).
2. Scheduler-service sends a REST GET request to the User-service to fetch all users with birthdays on the current date.
3. User-service returns a list of target users and their primary guild IDs.
4. Scheduler-service publishes a `birthday_notification` async event for each user.
5. Gateway consumes the event, formats a congratulatory embed, and sends it to the configured general channel of the respective guilds.

```mermaid
sequenceDiagram
    participant SS as Scheduler-service
    participant US as User-service
    participant MB as Message Broker
    participant G as Gateway
    participant D as Discord Channel
    SS->>SS: Cron Trigger (00:00 UTC)
    SS->>US: GET /api/users/birthdays?date=today
    US-->>SS: List of Users & Guilds
    loop For each User
        SS->>MB: Publish `birthday_notification`
    end
    MB-->>G: Consume Event
    G->>D: Send Birthday Embed
```

---

### Failure Flow

## Music Service Unavailable
1. User invokes the `/play <query>` command.
2. Gateway attempts to send a REST POST request to the Music-service.
3. The request times out or the Music-service returns a 503 Service Unavailable error.
4. Gateway's internal configured Retry mechanism attempts the request 2 more times.
5. Upon repeated failure, the Gateway's Circuit Breaker trips to the "Open" state to prevent cascading failures.
6. Gateway immediately responds to the user's interaction with an ephemeral error message: "The Music service is currently unavailable. Please try again later."
7. Circuit Breaker handles cooldowns before periodically allowing a test request to see if the Music-service has recovered.

```mermaid
sequenceDiagram
    participant U as User
    participant G as Gateway
    participant M as Music-service
    U->>G: /play <query>
    loop Retry (x3)
        G->>M: POST /api/music/play
        M--xG: Timeout / 503 Error
    end
    G->>G: Open Circuit Breaker
    G->>U: Ephemeral Error "Service Unavailable"
    Note over G: Subsequent requests fail fast<br/>until cooldown expires
```

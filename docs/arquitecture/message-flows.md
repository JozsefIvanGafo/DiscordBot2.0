# Message Flows

## Overview

This document describes the main interaction flows between the gateway and microservices in the system. It focuses on user actions, internal service communication, and system reactions.

---

## Play Song Flow

1. User clicks play button (or interaction)
2. Gateway receives the interaction event
3. Gateway validates user permissions
4. Gateway sends request to music-service (REST: POST /play)
5. Music-service processes the request and updates the queue/player
6. Music-service emits `track_started` event
7. Gateway receives the event
8. Gateway updates the music controller UI (embed/buttons)

---

## Pause / Resume Flow

1. User clicks pause/resume button
2. Gateway receives interaction
3. Gateway sends request to music-service (POST /pause or /resume)
4. Music-service updates player state
5. Gateway updates UI accordingly

---

## Skip Track Flow

1. User clicks skip button
2. Gateway receives interaction
3. Gateway sends request to music-service (POST /skip)
4. Music-service removes current track and selects next
5. Music-service emits `track_started` event
6. Gateway updates UI with new track

---

## Add to Queue Flow

1. User requests to add a song (URL or search)
2. Gateway receives input
3. Gateway sends request to music-service (POST /queue)
4. Music-service adds track to queue
5. Music-service emits `queue_updated` event
6. Gateway updates queue display in UI

---

## Volume Change Flow

1. User adjusts volume control
2. Gateway receives interaction
3. Gateway sends request to music-service (POST /volume)
4. Music-service updates volume level
5. Gateway updates UI if necessary

---

## Loop Mode Flow

1. User toggles loop mode
2. Gateway receives interaction
3. Gateway sends request to music-service (POST /loop)
4. Music-service updates loop state (track or queue)
5. Gateway updates UI to reflect loop mode

---

## Leave Voice Channel Flow

1. User clicks leave button
2. Gateway receives interaction
3. Gateway sends request to music-service (POST /leave)
4. Music-service disconnects from voice channel
5. Gateway updates UI (disable controls or reset player)

---

## Track Finished Flow (Async Event)

1. Music-service detects that the current track has finished
2. Music-service emits `track_finished` event
3. Gateway receives the event
4. Gateway updates UI
5. Gateway may trigger next track (optional behavior)

---

## Moderation Flow (Example: Kick User)

1. Admin triggers moderation command
2. Gateway validates admin permissions
3. Gateway sends request to moderation-service
4. Moderation-service applies action (kick/ban/mute)
5. Gateway confirms action to Discord

---

## Auto-Role Flow

1. User joins server
2. Gateway receives join event
3. Gateway requests role configuration from config-service
4. Gateway assigns roles to user

---

## Birthday Notification Flow

1. Scheduler-service runs periodic job
2. Scheduler queries user-service for today's birthdays
3. Scheduler triggers notification event
4. Gateway receives event
5. Gateway sends message to Discord channel

---

## Failure Flow (Music Service Unavailable)

1. User clicks play
2. Gateway sends request to music-service
3. Request fails (timeout or error)
4. Gateway retries request (if configured)
5. Circuit breaker may open after repeated failures
6. Gateway returns safe error message to user
7. UI reflects failure state
# Service Communication Design

## Overview

The system uses a hybrid communication model:

- Synchronous communication via REST APIs
- Asynchronous communication via events (future: Redis / RabbitMQ)

---

## REST Communication

Used for direct user actions.

### Gateway → Music Service

| Action        | Endpoint              |
|--------------|----------------------|
| Play         | POST /play           |
| Pause        | POST /pause          |
| Skip         | POST /skip           |
| Add to queue | POST /queue          |
| Volume       | POST /volume         |
| Leave VC     | POST /leave          |

---

## Event Communication

Used for internal updates.

### Events

- track_started
- track_finished
- queue_updated
- error_occurred

---

## Event Payload Example

```json
{
  "event": "track_started",
  "guild_id": "123",
  "track": {
    "title": "Song",
    "url": "..."
  }
}
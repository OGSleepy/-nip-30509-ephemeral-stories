# NIP-XX: Ephemeral Stories

`draft` `optional`

---

## Abstract

This NIP defines **kind `30509`** — Ephemeral Story — a short-lived, addressable event that allows users to share image or video content that automatically expires after a set duration (typically 24 hours), similar to "Stories" on other social platforms.

---

## Motivation

Nostr has no standard for ephemeral media posts. Users cannot currently share time-limited content (photos, short videos) in a way that:

1. Is semantically distinct from regular notes (kind `1`)
2. Self-expires and can be filtered by relays accordingly
3. Can be individually identified and addressed (e.g. for reactions or replies)

Kind `30311` (NIP-102, Live Activity) is sometimes misused for this purpose, but it is specifically defined for live stream metadata and carries incompatible semantics. This NIP defines a dedicated kind for ephemeral stories.

---

## Event Definition

**Kind:** `30509`

Kind `30509` is an **addressable (parameterized replaceable) event** per NIP-01 (kinds 30000–39999). Each story is a distinct addressable event identified by a unique `d` tag.

### Tags

| Tag | Req? | Description |
|-----|------|-------------|
| `d` | REQUIRED | Unique identifier for this story (e.g. a UUID or timestamp-based string). Required for addressable events. |
| `url` | REQUIRED | Direct URL to the media file (image or video). |
| `m` | REQUIRED | MIME type of the media (e.g. `image/jpeg`, `image/png`, `image/webp`, `video/mp4`, `video/webm`). |
| `expiration` | REQUIRED | Unix timestamp (seconds) when this story expires. MUST be set. Relays implementing NIP-40 MAY delete the event after this time. Clients MUST NOT display stories past their expiration timestamp. |
| `imeta` | RECOMMENDED | NIP-92 inline metadata for the media URL. Provides richer media info (dimensions, blurhash, size). |
| `thumb` | OPTIONAL | URL to a thumbnail image (useful for video stories). |
| `title` | OPTIONAL | Short display title for the story. |
| `summary` | OPTIONAL | Short text summary or caption. |
| `t` | OPTIONAL | Hashtags for discovery. |
| `location` | OPTIONAL | Geolocation string. |

The `.content` field MAY contain a caption or description as a plain-text string.

### Standard Expiry

The RECOMMENDED default expiry is **24 hours** from `created_at`:

```
expiration = created_at + 86400
```

Clients MAY allow users to set shorter durations (e.g. 1 hour, 6 hours). Durations longer than 24 hours are DISCOURAGED to preserve the ephemeral nature of stories.

---

## Example Event

```json
{
  "kind": 30509,
  "pubkey": "f7234bd4c1394dda46d09f35bd384dd30cc552ad5541990f98844fb06676e9ca",
  "created_at": 1700000000,
  "tags": [
    ["d", "story-1700000000-x7k2m9p"],
    ["url", "https://nostr.build/i/abc123.jpg"],
    ["m", "image/jpeg"],
    ["expiration", "1700086400"],
    ["imeta",
      "url https://nostr.build/i/abc123.jpg",
      "m image/jpeg",
      "dim 1080x1920",
      "blurhash LGF5]+Yk^6#M@-5c,1J5@[or[Q6."
    ],
    ["thumb", "https://nostr.build/t/abc123.jpg"],
    ["t", "photography"]
  ],
  "content": "Golden hour 🌅",
  "id": "...",
  "sig": "..."
}
```

### Video Story Example

```json
{
  "kind": 30509,
  "pubkey": "f7234bd4c1394dda46d09f35bd384dd30cc552ad5541990f98844fb06676e9ca",
  "created_at": 1700000000,
  "tags": [
    ["d", "story-1700000000-v8n3q2z"],
    ["url", "https://nostr.build/v/clip456.mp4"],
    ["m", "video/mp4"],
    ["expiration", "1700086400"],
    ["imeta",
      "url https://nostr.build/v/clip456.mp4",
      "m video/mp4",
      "dim 1080x1920"
    ],
    ["thumb", "https://nostr.build/t/clip456.jpg"]
  ],
  "content": "Check out this sunset!",
  "id": "...",
  "sig": "..."
}
```

---

## Client Behavior

### Displaying Stories

- Clients MUST filter out events where `expiration` tag value ≤ current unix time.
- Clients SHOULD display stories in a horizontally scrollable ring-style UI, grouped by author.
- Clients SHOULD track which stories have been viewed locally (this state is not published to relays).
- Clients SHOULD auto-advance through a user's multiple stories after a fixed duration (e.g. 5 seconds per story).

### Querying Stories

To fetch active stories, clients SHOULD query with a `since` filter to avoid fetching already-expired events:

```json
[
  "REQ",
  "<subscription_id>",
  {
    "kinds": [30509],
    "since": <current_unix_timestamp_minus_86400>
  }
]
```

### Addressing Stories

Because kind `30509` is addressable, a specific story can be referenced using an `a` tag:

```
a = "30509:<pubkey>:<d-tag-value>"
```

This enables reactions (kind `7`), replies (kind `1111` per NIP-22), and zaps (NIP-57) to target specific stories.

### Publishing Stories

When a client publishes a story:

1. A **unique `d` tag** MUST be generated (e.g. `story-<timestamp>-<random>`). Without this, publishing a second story would overwrite the first on supporting relays, since addressable events with the same `pubkey + kind + d` are treated as the same event.
2. The `expiration` tag MUST be set.
3. Clients SHOULD upload media to a NIP-96 compatible host before publishing the event.

---

## Relay Behavior

- Relays implementing **NIP-40** (Expiration Timestamp) SHOULD delete or stop serving kind `30509` events after their `expiration` timestamp.
- Relays MAY choose not to store kind `30509` events at all (since they are ephemeral), but SHOULD serve them while they are active.
- Because kind `30509` is addressable, relays MUST replace an existing event when a new one arrives with the same `pubkey + kind + d` combination and a newer `created_at`.

---

## Backwards Compatibility

This NIP introduces a new kind and does not modify any existing kinds or protocol flows. Clients and relays that do not implement this NIP will simply ignore kind `30509` events. No existing functionality is affected.

---

## Relationship to Other NIPs

| NIP | Relationship |
|-----|-------------|
| NIP-01 | Defines addressable events (kinds 30000–39999) and the `d` tag requirement |
| NIP-40 | Defines the `expiration` tag used for relay-side auto-deletion |
| NIP-92 | Defines `imeta` tag used for rich media metadata |
| NIP-96 | Recommended media hosting protocol for story uploads |
| NIP-94 | Kind `1063` File Metadata — a related but non-ephemeral alternative for file sharing |
| NIP-102 | Kind `30311` Live Activity — a distinct kind for live streams; NOT suitable for stories |

---

## Open Questions

- Should reactions/replies to expired stories be kept or also deleted?
- Should there be a standard story viewer app handler registered via NIP-89?
- Should a `duration` tag be defined to indicate video length for progress bar rendering?

---

## Reference Implementation

- [nostr-social (this app)](https://github.com/) — React/TypeScript client using `@nostrify/nostrify`

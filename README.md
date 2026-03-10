# 📸 NIP-XX: Ephemeral Stories — Kind `30509`

> A proposal for Instagram-style ephemeral stories on Nostr. Short-lived. Media-first. Fully decentralized.

---

## What is this?

This repo contains a draft NIP (Nostr Implementation Possibility) for **ephemeral stories** — image and video posts that automatically expire after 24 hours, similar to Instagram Stories, Snapchat, or WhatsApp Status.

**→ [Read the full proposal](./nip-ephemeral-stories.md)**

---

## Why a new Kind?

No existing Nostr kind fits this use case cleanly:

| Kind | What it's for | Why it doesn't work |
|------|--------------|---------------------|
| `1` | Text notes | No expiry, no media-first semantics |
| `1063` | File metadata (NIP-94) | Not ephemeral, not story-oriented |
| `30311` | Live Activity (NIP-102) | For live streams — wrong semantics entirely |
| **`30509`** | **Ephemeral Story** | ✅ This proposal |

---

## How it works

A kind `30509` event is an **addressable event** (NIP-01) with a mandatory `expiration` tag (NIP-40). Each story is uniquely identified by a `d` tag, supports rich media via `imeta` (NIP-92), and self-expires after 24 hours.

```json
{
  "kind": 30509,
  "tags": [
    ["d", "story-1700000000-x7k2m9p"],
    ["url", "https://nostr.build/i/abc123.jpg"],
    ["m", "image/jpeg"],
    ["expiration", "1700086400"],
    ["imeta", "url https://nostr.build/i/abc123.jpg", "m image/jpeg", "dim 1080x1920"]
  ],
  "content": "Golden hour 🌅"
}
```

Key design decisions:

- **`d` tag is required** — without it, publishing a second story overwrites your first on relays (addressable event rules)
- **`expiration` tag is required** — relays implementing NIP-40 can auto-delete; clients MUST NOT display expired stories
- **`imeta` tag is recommended** — enables thumbnails, dimensions, blurhash previews before full media loads
- **Reactions & replies are possible** — since events are addressable, they can be targeted with `a` tags

---

## Status

`draft` — seeking feedback before submitting to [nostr-protocol/nips](https://github.com/nostr-protocol/nips)

- [ ] Community feedback
- [ ] At least 2 client implementations
- [ ] At least 1 relay implementation
- [ ] Submit PR to nostr-protocol/nips

---

## Reference Implementation

This NIP was designed alongside a working Nostr social client built with React + TypeScript + [@nostrify/nostrify](https://github.com/soapbox-pub/nostrify). The implementation covers:

- Publishing kind `30509` events with correct `d` + `expiration` + `imeta` tags
- Subscribing to and rendering stories grouped by author
- 24h expiry enforcement on the client side
- Story viewer with progress bar, navigation, and tap zones

---

## Feedback welcome

If you're a client dev, relay operator, or just a Nostr power user with opinions — open an issue or reach out on Nostr.

Things especially worth debating:

- Should there be a standard NIP-89 handler registered for story viewers?
- Should a `duration` tag be defined for video length (useful for progress bars)?
- What should happen to reactions/replies pointing to an expired story event?

---

## Related NIPs

- [NIP-01](https://github.com/nostr-protocol/nips/blob/master/01.md) — Addressable events & `d` tag
- [NIP-40](https://github.com/nostr-protocol/nips/blob/master/40.md) — Expiration timestamp
- [NIP-92](https://github.com/nostr-protocol/nips/blob/master/92.md) — Inline media metadata (`imeta`)
- [NIP-96](https://github.com/nostr-protocol/nips/blob/master/96.md) — HTTP file storage (recommended upload method)
- [NIP-102](https://github.com/nostr-protocol/nips/blob/master/102.md) — Live Activity (kind `30311`, NOT for stories)

---

*Draft by [@you](https://nostr.com) · Contributions welcome · Not yet merged into nostr-protocol/nips*

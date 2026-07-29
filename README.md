# nowplaying

A page that shows whatever I'm currently playing in Apple Music:
**https://809345127.github.io/nowplaying/**

It's the link in my Feishu profile signature. Feishu renders its own small
preview inline in the signature; clicking through lands here, where the cover
art gets the space it deserves.

## How it fits together

```
Apple Music  ──broadcast──>  local daemon  ──push──>  Cloudflare Worker
 (this Mac)                   (this Mac)                 (relay)
                                   │                        │
                                   │ WebSocket              │ fetch
                                   ▼                        ▼
                             Feishu signature          this page
                              (small preview)         (big cover art)
```

- **local daemon** — a Go program listening for macOS's `com.apple.Music.playerInfo`
  broadcast, so it learns about track changes the instant they happen without
  polling Music.app or needing Automation permission. Source lives outside this
  repo (`~/GolandProjects/feishu-nowplaying`).
- **Cloudflare Worker** — holds the current track in a Durable Object. Writes are
  authenticated with a shared secret; reads are open to this page's origin only.
- **this page** — one self-contained `index.html`, no build step. Polls the relay
  every 3 seconds while the tab is visible, and derives its entire palette from
  the album artwork, so every song gives the page a different look.

## Behaviour worth knowing

- **Nothing playing** → a quiet "现在没在听歌".
- **Mac asleep or offline** → after 3 minutes with no heartbeat, the page says so
  rather than leaving an hours-old song on screen pretending to be live.
- **Relay unreachable** → the last known track stays on screen. A network blip
  should not blank the page.

## Editing

Change `index.html` and push; GitHub Pages picks it up. Note that Pages serves
with a 10-minute cache and **ignores query strings**, so a `?v=2` cache-buster
does nothing — just wait it out.

The data endpoint is the `API` constant at the top of the page's script.

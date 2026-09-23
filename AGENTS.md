# AGENTS.md — roon-mpris/

Parent: `~/AGENTS.md` | Full reference: `CLAUDE.md`

## OVERVIEW

Node.js extension connecting Roon to Linux MPRIS D-Bus. Each Roon zone exposed as its own MPRIS player for media-key control via `playerctl`.

## STRUCTURE

```
roon-mpris/
├── index.js       # Single-file application (~480 lines)
├── package.json   # Dependencies + CLI bin
├── CLAUDE.md      # Full reference (architecture, known issues)
└── README.md      # Usage docs
```

## ARCHITECTURE

Adapter-per-entity pattern:
- `zonePlayerMap`: `Map<zone_id, PlayerContext>` tracks all zone players
- `sanitizeForDBus()`: converts zone names to valid D-Bus names
- `updatePlayerFromZone()`: syncs Roon zone state → MPRIS metadata
- `setupPlayerEvents()`: binds MPRIS events → Roon controls

### Event Flow

**Roon → MPRIS:** `subscribe_zones` callback creates/updates/destroys players.
**MPRIS → Roon:** `player.on('playpause'|'stop'|'next'|'previous')` calls `RoonApiTransport.control()`.

## CONVENTIONS

- D-Bus naming: `Living Room` → `org.mpris.MediaPlayer2.roon_Living_Room`
- Config stored at `~/.config/roon-mpris/`

## ANTI-PATTERNS

- Uses internal `player._bus.disconnect()` — mpris-service lacks public destroy method
- `canPlay` commented out to suppress Ubuntu dock widget

## COMMANDS

```bash
npm install
npm start                    # Normal mode
node index.js --pause-all    # Pause all zones and exit
node index.js --host <ip>    # Direct connection (bypass multicast)
node index.js --log all      # Debug Roon API
busctl --user list | grep MediaPlayer2   # Verify registration
playerctl -p roon_Living_Room play       # Control specific zone
```

## DEPENDENCIES

- `node-roon-api` / `node-roon-api-transport` / `node-roon-api-settings`
- `mpris-service` (D-Bus implementation)
- `yargs` (CLI parsing)

# Attune

Track attunement/key/raid-access progression for your characters, share progress with guildmates, and export data for external tools.

Originally by **Cixi** (Remulos). This repository contains a port used by private server communities.

## What the addon does

Attune keeps a local database of characters you’ve seen (including yourself) and records:

- Character metadata (name, class, race, level, guild)
- Attunement progress per “attune” entry
- Step completion for quest/item/rep requirements

It provides three main workflows:

1. **Personal progress view**: a tree of attunements and their steps.
2. **Survey**: ask guild/party/raid members running Attune to send you their progress.
3. **Export**: generate a compact encoded string of the current dataset (self / guild / last survey / etc).

## Repository layout

- `Attune.toc` — addon metadata (interface version, load order, saved variables)
- `Attune.lua` — core addon logic (UI, scanning, comms, export/sync)
- `AttuneData.lua` — the “database” describing attunements and steps
- `Locales/` — translations (AceLocale)
- `Libs/` — bundled Ace3 + supporting libraries

## Installation

1. Copy the `Attune` folder into your WoW client’s AddOns folder:

   - `World of Warcraft/Interface/AddOns/Attune`

2. Restart the game (or `/reload`).
3. Enable **Attune** at the character selection “AddOns” button.

## In-game usage

Open the main window:

- `/attune`

From the UI:

- **Survey** collects data from other players using the addon
- **Export** opens a window with an encoded export string
- **Results** toggles between your attunement tree and survey/guild results
- **Raid Planner** helps build raid rosters from known characters

## Slash commands

The addon supports a small command set (see `Locales/enUS.lua` help strings and `Attune.lua` slash handler):

- `/attune` — open the UI
- `/attune info` — addon info
- `/attune survey` — run a guild survey
- `/attune silentsurvey` — same as survey, fewer chat messages
- `/attune sync` — request a sync from your target
- `/attune raid` / `/attune raidplanner` / `/attune planner` — open the raid planner
- `/attune <name>` — survey a specific player

## How surveying works

Surveying is done via addon messages (AceComm). Conceptually:

1. You send a survey request (guild/whisper).
2. Recipients reply with:
   - A **TOON** “meta” message (class/race/level/guild/version/etc)
   - Multiple **DONE** messages (completed step IDs)
   - A closing **OVER** message
3. Your client merges that into `Attune_DB.toons`.

The addon includes cooldown/anti-spam protections and avoids accepting “spoofed” survey messages about other players.

## How sync works (one-to-one)

Sync is a consent-based exchange between you and your current target to merge datasets (useful for alt collections or rebuilding after a reset).

## Export format

Export generates a compact encoded string containing the selected dataset:

- your character only
- last survey
- guild dataset
- all known characters
- (optionally) all your alts

This is intended for external tools/websites or for sharing.

## Saved variables

Attune stores state in the following saved variables (see `Attune.toc`):

- `Attune_DB` (account-wide)
- `AttuneLastViewed`, `TreeExpandStatus`, `AttuneCompletedQuests` (per character)

Important fields inside `Attune_DB` include:

- `toons` — all known characters and their progress
- `survey` — last-survey bookkeeping
- settings for UI, minimap button, sync/survey behavior

## Data model (how attunements are defined)

All attunement content is defined in `AttuneData.lua`.

### Attunes

`Attune_Data.attunes` contains “tracked attunements” that have step chains.

Each entry includes fields like:

- `ID` (string)
- `NAME`, `EXPAC`, `GROUP`, `FACTION`
- `ICON`, `DESC`
- `ATTLOC` / `TYPE` (used for location/detection logic)
- `GROUPSIZE`, `SHOWRAIDPLANNER` (raid planner integration)

### Steps

`Attune_Data.steps` contains the graph of requirements for each attune.

Important fields:

- `ID_ATTUNE` — which attune this step belongs to
- `ID` — step ID unique within that attune
- `TYPE` — `Quest`, `Pick Up`, `Turn In`, `Kill`, `Item`, `Rep`, `Level`, `Interact`, `End`
- `FOLLOWS` — dependency list (e.g. `"40&50"`)
- `STAGE` — used for layout in the UI
- `ID_WOWHEAD` — numeric identifier (quest ID, item ID, NPC ID, rep ID, etc)
- optional `COUNT` for item collection steps

The core logic uses this table to:

- decide what is “next”
- compute completion percentage
- draw the step nodes and dependency lines

### Raids with no attunement

Some raids do not have attunement chains on certain servers/patches. These can be represented as “auto-attuned” entries.

If you want those entries to show in the main tree UI, they must have the same metadata as regular attunes (`EXPAC`, `GROUP`, `FACTION`, `ICON`, `DESC`).

## Localization

All user-facing strings are in `Locales/*.lua` and loaded via AceLocale.

- If a key is missing in your locale, it falls back to `enUS`.

## Troubleshooting

- Addon not showing in the AddOns list:
  - check the folder name is exactly `Attune`
  - check the path is `Interface/AddOns/Attune/Attune.toc`
  - ensure the `.toc` `## Interface:` matches your client build
- UI opens but no data:
  - progress is built from quest log/bags/reputation scanning; some servers may differ
  - use **Survey** to populate guild datasets

## Community / contact

- Discord: https://discord.gg/MpfDeBZ

## External tools

Historically, exports were uploaded to:

- https://warcraftratings.com/attune

This repository does not require that site to function; export strings can be used by any compatible tool.



# Conformance Language
* `SHALL`:
  * an absolute requirement for all implementations
* `SHALL NOT`:
  * an absolute prohibition against inclusion for all implementations
* `SHOULD`/`SHOULD NOT`:
  * A best practice or recommendation to be considered by implementers within the context of their particular implementation;
    there may be valid reasons to ignore an item, but the full implications must be understood and carefully weighed before choosing a different course
* `MAY`:
  * This is truly optional language for an implementation; can be included or omitted as the implementer decides with no implications

# Bounce

All bounce packets put the custom data in the `data` tag, and target the appropriate tag in the `tags` field
ex.
```json
[{
  "cmd": "Bounce",
  "tags": ["DeathLink"],
  "data": {
    "time": 123,  // unix tiemstamp
    "cause": "The player Player1 died.",  // flavor text; omit key if missing
    "source": "Player1"  // slot name; mandatory
  }
}]
```

### DeathLink

type: Bounce
tag: `DeathLink`

data:
* `time`: int Unix timestamp of packet sent
  * SHALL be included
  * SHALL be used in the client to ignore a client's own packets reflected back
  * MAY ignore any packets older than the previous send or only packets that equal timestamp exactly
* `cause`: string flavor text about the deaths to display to player
  * SHALL be a string
  * MAY be omitted
  * SHALL include the player's slot name or in-game name as part of the message so it can be displayed as-is
  * MAY be used if present to display to the player
* `source`: string name of the player who died
  * SHALL be included
  * SHALL be a string
  * SHALL be either the player's slot name or in-game name
  * MAY be used to display a generic death text, ex. f"{source} has died."

#### Grouped DeathLink

Grouped DeathLinks look and act like normal DeathLink packets, but have following differences
* The user SHALL be allowed to define an arbitrary group name, either in client or in options
* The group name SHALL be case sensitive
* The group name SHALL be appended to the DeathLink tag when connecting and when sending deathlinks
***TODO: confirm this***
* A client SHALL NOT connect as both base DeathLink and a group
* A client MAY connect as multiple groups
  * If a client is in multiple groups it SHALL include all in the tags of one DeathLink packet and not send multiple


### DamageLink

type: Bounce
tag: `SharedDamage`

data:
* `time`: int Unix timestamp of packet sent
  * SHALL be included
  * SHALL be used in the client to ignore a client's own packets reflected back  // TODO: lx5's notes say useful if you want to ignore by time
  * MAY ignore any packets older than the previous send or only packets that equal timestamp exactly
* `uuid`: string UUID identifier for the client sending the packet
  * ??? be included
  * MAY be used in the client to ignore a client's own packets while allowing packets from other clients on the same slot
* `damage_points`: int declaring the severity of the damage
  * SHALL be included
  * Clients SHALL define a ratio of damage_points per in game health, even if that is 1:1
    * Clients MAY use 80-160 damage_points per full health bar as a benchmark for deciding the ratio
  * Clients MAY define a limit for incoming damage

### Grouped DamageLink

Grouped DamageLinks look and act like normal DamageLink packets, but have following differences
* The user SHALL be allowed to define an arbitrary group name, either in client or in options
* The group name SHALL be case sensitive
* The group name SHALL be appended to the SharedDamage tag when connecting and when sending DamageLinks
***TODO: confirm this***
* A client SHALL NOT connect as both base DamageLink and a group
* A client MAY connect as multiple groups
  * If a client is in multiple groups it SHALL include all in the tags of one DamageLink packet and not send multiple


### KnockbackLink

type: Bounce
tag: `KnockbackLink`

data:
* `source`: string name of the player who died
  * SHALL be included
  * SHALL be a string
  * SHALL either be the player's slot name or in-game name
  * MAY be used to display a generic death text
* `uuid`: string UUID identifier for the client sending the packet
  * ??? be included
  * MAY be used in the client to ignore a client's own packets while allowing packets from other clients on the same slot
* `cause`: string flavor text about the knockback to display to the player
  * SHALL be included
  * MAY be used to display to the player
* `time`: int Unix timestamp of packet sent
  * SHALL be included
  * SHALL be used in the client to ignore a client's own packets reflected back
  * MAY ignore any packets older than the previous send or only packets that equal timestamp exactly
* `value`: object with the following definition
  * `x`: float value of knockback; + is east/left, - is west/right
  * `y`: float value of knockback; + is up, - is down
  * `z`: float value of knockback; + is south, - is north
  * All value attributes SHALL be included
  * All values not used when sending SHALL be set to 0
  * All values not used when receiving SHALL be ignored
  * All values SHALL be calculated as the change in velocity of the player
  * Clients MAY include some scaling multiplier to normalize knockbacks
    * Clients MAY use 40 on the wire as a baseline for generic knockbacks and 100 as a baseline for large knockbacks like a bomb
  * Clients MAY cap incoming knockback velocity
  * Clients MAY batch outgoing knockbacks


### Ring Link

type: Bounce
tag: `RingLink`

data:
* `time`: int Unix timestamp of packet sent
  * SHALL be included
* `source`: int unique-per-connection identifier
  * SHALL be included
  * SHALL be unique per connection
  * SHALL be used to ignore a client's own packets while allowing packets from other clients on the same slot
  * MAY use the shared UUID
* `amount`: int value of rings gained or lost
  * SHALL be included
  * Clients SHOULD increase the appropriate attribute on positive amounts and decrease on negative amounts but if they do not honor
    increase or decrease they SHALL ignore the packet entirely
  * Clients MAY have a multiplier to normalize changes
  * Clients MAY ignore amount changes if the player is not in a state to receive them (main menu, death, etc.)


### Trap Link

type: Bounce
tag: `TrapLink`

data:
* `time`: int Unix timestamp of packet sent
  * SHALL be included
* `source`: string slot name of sending player
  * SHALL be included
  * SHALL be used to ignore same-slot packets to not duplicate sent items
    * TODO: MAY when local item handling? unsure
    * TODO: do other client's need to filter out same source?
* `trap_name`: string name of trap
  * SHALL be included
  * MAY be converted to another known trap name when receiving if thematically correct
    * Known trap names and how other games convert are documented in a
      [spreadsheet](https://docs.google.com/spreadsheets/d/1yoNilAzT5pSU9c2hYK7f2wHAe9GiWDiHFZz8eMe1oeQ/edit?gid=811965759#gid=811965759)
  * Linked Traps MAY be queued

# DataStorage

// TODO: flush out

placeholders: since datastorage is shared across all clients datastorage keys will usually have some placeholders for values that differentiate clients, slots, etc.
When written in this doc they will use python f-string format, example: the key f"{team}:{slot}:PlaceholderLink" would be `0:3:PlaceholderLink` for slot 3 on team 0
* `slot_name`: player name of the connected slot
* `slot`: slot id/number of the connected slot
* `team`: team id/number of the connected slot
* ...


### EnergyLink

type: DataStorage
key: f"EnergyLink{team}"

value: int
  * The key SHALL be increased by the `add` operation
  * The default of `0` SHALL be included
  * The key SHALL be decreased by combining the `add` operation with a negative values and a `max` operation of `0`
  * The client SHALL react to the `SetReply` message and only apply the effect in game based on the difference between `original_value` and `value`


### Gifting

// TODO: flesh this out
https://github.com/agilbert1412/Archipelago.Gifting.Net/blob/main/Documentation/Gifting%20API.md

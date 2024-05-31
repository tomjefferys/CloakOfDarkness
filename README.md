# CloakOfDarkness
Cloak of Darkness ported to TIFT

Playable here: https://tomjefferys.github.io/CloakOfDarkness/

```yaml
--- # src/game.yaml
game: Cloak of Darkness
author: Presto Turnip
version: 1.0.0
gameId: 58379ffe-9ec5-4317-bdd2-1b79c171a2ec
description: >
  A port of Cloak of Darkness to function as a demonstration of TIFT
options:
  - useDefaultVerbs
introText: > 
  Hurrying through the damp autumn evening, 
    thankful to have your favourite cloak to keep the rain off,
    you finally approach the grand opera house.
    The streets are are strangely deserted,
    but this is not really unexpected in a simple demo game.
    {{br}}
    **Cloak of Darkness**
    {{br}}
    A basic IF demonstration.
maxScore: 2
beforeGame():
  - print(introText)

--- # src/verbs/hang.yaml
verb: hang
tags:
  - transitive
attributes:
  - "on"
contexts:
  - inventory
  - wearing
actions:
  hang($hangable).on($hanger):
    - moveItemTo(hangable, hanger)

--- # src/rooms/bar.yaml
room: bar
name: Bar
description: >
  A rough looking bar, It is completely empty except for a layer of sawdust covering the floor.
  It looks like some sort of message has been written in the sawdust.
tags:
  - dark
exits:
  north: foyer
blunderCount: 0
onAddChild(entity): 
  when: isAtLocation(cloak, entity)
  then: 
    - setTag(this, 'dark')
    - openExit(bar, 'east', bar)
    - openExit(bar, 'south', bar)
    - openExit(bar, 'west', bar)
  otherwise: 
    - delTag(this, 'dark')
    - closeExit(bar, 'east')
    - closeExit(bar, 'south')
    - closeExit(bar, 'west')
before:
  go($direction):
    unless: direction == 'north'
    do: 
       - print("Blundering around in the dark isn't a good idea!")
       - blunderCount = blunderCount + 1

--- # src/rooms/foyer.yaml
room: foyer
name: Foyer of the Opera House
description: >
  You are standing in the foyer of the opera house.
  The palatial space is lit from above by large chandeliers.
  The entrance from the street is to the north, there doors to the south and west.
tags:
  - start
exits:
  south: bar
  west: cloakroom
  north: outside
leaveMessage: >
  You've only just arrived, and besides, the weather outside seems to be getting worse.
before:
  go(north): leaveMessage

--- # src/rooms/cloakroom.yaml
room: cloakroom
name: Cloakroom
description: >
  The small cloakroom has seen better days.
  Marks on the walls show where hooks used to be attached, now all removed except for one.
  The exit is a door to the east.
exits:
  east: foyer

--- # src/items/cloak.yaml
item: cloak
name: velvet cloak
description: >
  A cloak of purest black, a little damp from the rain, its darkness seems to suck in all the light.
tags:
  - carryable
  - worn
verbs:
  - hang
dropMessage: >
  This isn't the best place to leave a smart velvet cloak
before:
  drop(this): dropMessage
  
--- # src/items/hook.yaml
item: hook
name: small brass hook
description: >
  A small brass hook
  {{#cloakOnHook}} with a cloak hanging on it.{{/cloakOnHook}}
  {{^cloakOnHook}} screwed to the wall. {{/cloakOnHook}}
cloakOnHook(): 
  - onHook = isAtLocation(cloak, hook)
  - return(onHook)
location: cloakroom
adposition: hanging on
verbs:
  -  hang.on
after:
  hang(cloak).on(this):
    once: score(1)

--- # src/items/message.yaml
item: message
name: message
location: bar
messages:
  win: >
    The message, neatly marked in the sawdust, reads...{{br}}
    You have won.
  lose: >
    The message has been carelessly trampled, making it difficult to read.
    You can just distinguish the words...{{br}}
    You have lost.
verbs: ["examine"]
before:
  examine(this):
    when: bar.blunderCount <= 2
    then: 
      - once: score(1)
      - gameOver(messages.win)
    else: gameOver(messages.lose)
```
# moonedit operator guide

moonedit edits the world with sentences typed after `/mo`. A sentence says
what to do and where; flags, each starting with `-`, say how. Tab completion
follows the grammar, so pressing Tab after any word shows what may come next,
including block, biome, clipboard, mark and archive names. Several sentences
can share one line, separated by `;`, and each starts once the one before it
has finished, so a sentence always sees what the earlier ones did:

```text
/mo select sphere 12 at target ; fill stone ; undo
/mo load fang-manor as #fang ; paste #fang at #site down 1 ; move up 1
```

A sentence that fails stops the rest of its line, which says so.

The [interactive manual](./) holds the same reference with a
command builder, which builds any sentence (brushes included) and says what it
does, and diagrams of what the flags do.

Edits run as background jobs: the server keeps ticking, progress shows on the
action bar, and the result (blocks changed, time taken) arrives in chat. Every
edit can be undone, even after a restart. Your sentences run in the order you
send them: one typed while an earlier one is still running waits for it, and
`target` in it still means what you looked at when you sent it. `jobs` and
`cancel` on their own answer at once.

moonedit is for server operators only: players need operator level 2 (the
server's `access.operatorLevel` setting). A permission mod such as FTB Ranks
can take moonedit away from an operator through the `moonedit.use` node, but
can never give it to anyone else; brush and wand items do nothing in other
hands. The server console and RCON can run every sentence, typed without the
slash. They stand at world spawn facing south (unless `/execute rotated` turns
them) and have no view, so they use coordinates instead of `target`.

## Selecting

| Sentence | Selects |
| --- | --- |
| `select sphere 12` | a sphere of radius 12 (25 blocks across) |
| `select cylinder 8 20` | a cylinder of radius 8, 20 high, rising from the anchor (1 high without a height) |
| `select ellipsoid 12x6x12` | an ellipsoid with these radii |
| `select box 10x4x10` | a box 10 wide, 4 high, 10 deep, centered horizontally, rising from the anchor; `select box 8` is a cube |
| `select from <place> to <place>` | the box between two places |
| `from [place]`, then `to [place]` | the box between two corners set one at a time (default: where you stand) |
| `select chunk`, `select chunks 2` | the chunk around the anchor, or that many chunks around it each way, full world height |

`from` and `to` are the two corners of a box, set whenever you like: stand at
one corner and say `from`, fly to the other (in spectator through the ground
if need be) and say `to`. Each takes a place too (`to target`, `from #gate`,
`to 100 -20 64`, `from target down 3`). The wand sets the same two corners by
clicking.

A shape rises from, or centers on, where you stand unless you name another
place with `at` (see [Places](#places)): `select sphere 12 at target`,
`select box 9 at #gate up 2`.

Selections combine: `select add <shape>` unites, `select cut <shape>` removes
and `select keep <shape>` intersects. Box selections also change with
`select grow 3 [directions]` and `select shrink 2 [directions]` (every side
unless directions are named) and `select extend up|down to sky|bedrock|surface|<y>`;
any selection moves with `select shift north 10`. `select trim` shrinks the
selection to the box around the blocks in it that are not air.
`select save as #name` and `select #name` keep and recall selections,
`select clear` drops it and `info` describes it.

## Places

A place is where you stand (`here`), the block you look at (`target`, up to
256 blocks away), a mark (`#gate`) or coordinates (`100 64 -20`, with `~` for
values relative to where you stand), followed by any nudges: a direction and a
number of blocks, 1 unless told. `#gate up 2`, `target down`, `here ahead 5`
and `100 64 -20 north 3 up 1` are all places, and nudges alone start from where
you stand (`ahead 5` is five blocks the way you face).

Directions are `up`, `down`, `north`, `south`, `east`, `west`, and relative to
where you face: `ahead`, `back`, `left`, `right`. North, up and the rest are
the world's directions wherever they appear.

`from`, `to`, `select from … to …` and `move to <place>` take their place as it is.
Everywhere else a place is optional and comes after `at`; leaving it out means
where you stand, except for `mark`, which names the block you look at:

| Sentence | Where |
| --- | --- |
| `select <shape> [at <place>]` | where the shape rises from or centers on |
| `mark #gate [at <place>]` | the place to name |
| `paste [#name] [at <place>]` | where the copy point lands |
| `copy [as #name] [at <place>]` | the copy point, where pastes will land |
| `stroke at <place>` | where the console's brush lands |

So `paste #fang at here down 1` pastes one block lower than usual,
`paste at target up` puts the copy point on top of the block you look at, and
`mark #door at target up` names the air above it.

## Blocks and materials

A block is written as in vanilla commands: `stone`, `oak_stairs[facing=north]`,
`create:andesite_casing`. A tag such as `#minecraft:logs` means any of its
blocks. A material mixes blocks by percentage: `stone 70%, andesite 30%`;
entries without a percentage share what is left. Mixes are reproducible: the
same edit on the same place picks the same blocks.

## Flags

Flags come after the rest of the sentence, may repeat and apply in the order
written. Tab completion offers only the flags a sentence takes, and a flag a
sentence does not take is refused rather than ignored.

| Flag | Means | Taken by |
| --- | --- | --- |
| `-where <filter>` | only blocks that match take part | the edits, `distribution`, the clipboard sentences |
| `-skip <filter>` | blocks that match are left out (`-where not …`) | the same |
| `-onto <filter>` | only destination blocks that match may be overwritten | `paste`, `stack` |
| `-swap <a> to <b>, …` | a clipboard's blocks change on the way through | the clipboard sentences |
| `-drained` | water and water plants become air, waterlogged blocks dry | the clipboard sentences |
| `-rotate <degrees>` | the clipboard turns clockwise seen from above, about its copy point, by a multiple of 90 (negative turns the other way) | the clipboard sentences |
| `-flip x\|y\|z` | the clipboard mirrors across its copy point; `-flip y` turns it upside down | the clipboard sentences |
| `-shift <direction> [blocks]` | the clipboard's blocks move away from its copy point, 1 block unless told | the clipboard sentences |
| `-with biomes` | the copied biomes are placed too | `paste`, `stack` |
| `-without entities` | entities stay behind | the clipboard sentences |
| `-whole` | keep the whole selection instead of trimming it | `copy`, `cut` |
| `-depth <n>` | how many blocks of air above the surface an overlay fills, up to 8 | `overlay` |
| `-soft <n>` | fade the edit out over the outer n blocks, up to 256; `-soft 0` keeps a hard edge | the terrain edits |
| `-replace`, `-next` | overwrite a taken archive name, or take the next free one | `save` |

The edits are the sentences under [Editing the selection](#editing-the-selection)
and [Shaping terrain](#shaping-terrain), except `biome` and `regen`, which take
no flags. The clipboard sentences are `copy`, `cut`, `paste`, `stack`, `move`,
`save` and `load`, and all of them take the same steps: `-where`, `-skip`,
`-swap`, `-drained`, `-rotate`, `-flip` and `-shift` form a pipeline the
clipboard's blocks pass through, left to right:
`paste -skip stone -swap oak_planks to stone` leaves out the stone the
clipboard held and turns its planks into stone, while
`paste -swap oak_planks to stone -skip stone` leaves out both. A block left
out is not part of the copy: a paste leaves the destination as it is there, a
move leaves it where it was, and a save does not write it. Entities come along
when the block they are in, or one beside, above or below it, stays part of
the copy, so `cut -where stone` leaves the cows on the grass alone. A list and
a repeated flag mean the same: `-skip water, kelp` is `-skip water -skip kelp`.

Turns and shifts follow the same order. A turn turns everything before it,
shifts included; a shift goes the world's way whatever came before, so
`-rotate 90 -shift north 2` lands two blocks north of the turned paste, while
`-shift north 2 -rotate 90` turns that shift too and lands two blocks east. A
`copy`, `cut` or `load` keeps its steps with the clipboard (load a build
`-shift down 1` and every paste of it lands one lower), a `save` writes them
into the file and leaves the clipboard as it was, and a `paste`, `move` or
`stack` applies them to what it places.

On edits that change the world in place (fill, swap, smooth and the rest),
`-where` and `-skip` narrow which blocks change, in any order:
`clear -where water -skip below 40`.

### Filters

| Filter | Matches |
| --- | --- |
| `stone`, `#minecraft:logs`, `oak_stairs[half=top]` | those blocks (all states unless properties are given) |
| `air`, `solid`, `liquid`, `natural` | air; blocks that stop movement; anything holding a fluid, waterlogged blocks included; terrain (stone, soils, sand, gravel, ores, snow, ice, terracotta, water, lava, clay, mud) |
| `exposed` | blocks touching a see-through block on any face |
| `surface` | non-air blocks with air above |
| `above 64`, `below 30` | by height |
| `near water 3` | within 3 blocks of water (up to 8) |
| `steep`, `steep 4` | on a slope: beside the block, the ground falls away at least 2 (or 4, up to 8) blocks within one step; the crests and faces of cliffs match, a one-block step does not |
| `not`, `and`, `or`, `( … )` | combinations; `and` binds tighter than `or`; in flags a comma is `or` |

Filters on the world read it as it was before the edit. A clipboard's blocks
are filtered by what they are, so `exposed`, `surface`, `above`, `below`,
`near` and `steep` apply only to the world.

## Editing the selection

| Sentence | Does |
| --- | --- |
| `fill <material>` | sets every selected block |
| `clear` | sets air |
| `drain` | turns water and water plants into air and dries waterlogged blocks; lava stays |
| `swap dirt to grass_block, stone to deepslate` | replaces blocks; a target without properties keeps the shared ones, so `swap oak_stairs to spruce_stairs` keeps facing and half |
| `walls <material>`, `outline <material>` | the selection's sides, or its whole shell |
| `hollow [thickness]` | empties the inside, keeping a shell (1 thick unless told) |
| `overlay <material> [-depth 2]` | fills the air above the surface |
| `biome <biome>` | sets biome cells (4x4x4 blocks each; a cell changes when its center is selected) |
| `regen` | regenerates the selection with the world's generator, biomes included |
| `count [-where <filter>]` | counts the blocks that match, or all that are not air |
| `distribution` | lists the selected blocks by kind, most common first |

`count` and `distribution` report without changing anything.

Edits change blocks without block updates: stairs do not reshape, sand does
not fall, water does not flow, machines are not triggered and nothing drops.
Block entities (chests, machines, signs) are created and removed properly and
keep their contents when only their state changes. Light, heightmaps, villager
workstations and what players see are always updated.

## Shaping terrain

Terrain edits read the height of the ground in every column, work out new
heights and rebuild the columns that change: raised ground grows from below
its top layers, so grass stays on top and dirt stays under it; lowered ground
is cleared to air, or to water on a lake bed; flowers and grass move with the
ground. Each is one undoable edit, and each works on the selection or as a
brush.

| Sentence | Does |
| --- | --- |
| `smooth [passes]` | blurs the ground; more passes smooth more (1 unless told, up to 64) |
| `raise [blocks]`, `lower [blocks]` | lifts or sinks the ground (1 block unless told, up to 256) |
| `flatten [to <height>]` | levels the ground to a height; without one, to the height where a brush stroke lands, or the selection's average |
| `terrace [step]` | cuts slopes into flat benches every `step` blocks (4 unless told, 2 to 64) |
| `roughen [blocks]` | adds natural-looking bumps and hollows up to that many blocks high or deep (2 unless told, up to 32) |
| `erode [strength]` | rain wears gullies into slopes and leaves what it carries on flatter ground below; higher strength rains longer (1 unless told, up to 16) |
| `weather [passes]` | crumbles cliffs and spikes into slopes no steeper than one block per block (1 unless told, up to 16) |
| `sharpen [strength]` | the opposite of smooth: creases deepen and crests stand out (1 unless told, up to 8) |
| `stretch <factor>` | scales the relief about the average height, by a factor from -4 to 4: `stretch 2` doubles it, `stretch 0.5` halves it, `stretch -1` turns hills into hollows |
| `reshape peaks`, `reshape domes`, `reshape mesas` | bends the relief between the lowest and highest ground: wide valleys under sharp peaks, broad rounded hills, or flat tops and floors joined by steep sides |
| `ridge [strength]` | raises sharp crests along the lines that climb toward each summit, never above it (1 unless told, up to 16) |
| `stream [depth]` | cuts a channel downhill from where a brush stroke lands (the highest ground for a selection) and fills it with water (2 deep unless told, up to 16) |
| `mountain [height]` | raises a craggy mountain over the middle of the selection or brush, its summit exactly that high (16 unless told, up to 256) |

The ground is the highest natural block of each column that is not a liquid:
stone, soils, sand, gravel, snow and the like; logs, leaves and builds are not
ground, so smoothing under a tree moves the soil, not the tree. `-where` and
`-skip` narrow what counts as ground, which also decides where an edit
applies: `flatten to 80 -where above 80` cuts every peak off at 80,
`flatten to 64 -where below 64` fills hollows up to 64, and
`smooth -where sand` smooths a beach without touching the grass behind it. A
column whose top is cut off by the selection or the filter, with more ground
right above it, is left alone rather than rebuilt with a gap. Raised ground
stops at the selection's top. One terrain edit covers at most 2048x2048
columns, and `erode` at most 1024x1024 at strength 4.

`-soft <n>` fades an edit out over the outer n blocks of the selection, so
`raise 6 -soft 12` lifts a plateau whose sides blend into the land around it.
Without it, selections have hard edges; brushes fade from their center to
their rim (see [Brushes and the wand](#brushes-and-the-wand)).

A stream runs the way water would, cutting through rises no higher than its
depth, until it reaches standing water, a hollow it cannot leave or the edge
of the selection or brush. Its bed deepens toward its mouth and never climbs,
its water stays a block below its banks, and its bed is soil rather than
grass; `drain` afterwards leaves a dry gully. A mountain fills its whole
footprint, rising from nothing at the rim; its crags grow rougher toward the
summit, and repeating it gives a different peak, since every stroke has its
own seed.

Layering materials by height and slope uses the ordinary edits with filters:
`swap grass_block to stone -where steep 3` bares the rock on cliffs,
`overlay snow -where above 150 -skip steep` snows the high ground but not its
cliffs, and `swap grass_block to sand -where below 64 and near water 3` makes
beaches.

## Clipboards

| Sentence | Does |
| --- | --- |
| `copy [as #name] [at <place>]` | copies the selection with block entities, biomes and entities; the copy point is the place, or where you stand |
| `cut [as #name] [at <place>]` | copies, then clears what it copied and removes its entities |
| `paste [#name] [at <place>]` | places a copy so the copy point lands at the place (default: where you stand) |
| `move <direction> [distance]` | moves the selection's blocks, entities and biomes (1 block unless told) |
| `move to <place>` | moves the selection so its lowest north-west corner lands at the place |
| `move from <place> to <place>` | moves the selection by the step from one place to the other |
| `stack <count> [direction]` | repeats the selection side by side, the way you face unless told (up to 256 copies) |
| `info [#name]` | describes the selection and your clipboard, or the named clipboard: its size and where its copy point is |

A paste puts the copy point on the place, so a clipboard lands the way it sat
around whoever copied it. Copy, cut, load and paste messages and `info` say
where the copy point lies, measured from the bottom center of the clipboard's
box. To choose it, copy `at` a place, such as a corner you marked; to move a
clipboard's blocks away from it for good, load or copy it with `-shift`.

A paste, a move and a stack select what they placed: the paste, the moved
blocks at their new place, or the source with all its copies. So right after a
paste, `move down 1` lowers it and `undo` takes the paste back; on one line,
`paste #fang at #site ; move down 1`. Moving keeps whatever the paste replaced
replaced, while undoing and pasting again lower restores it first:
`undo ; paste #fang at #site down 1`. Undo and redo put the selection back too,
as long as you have not selected something else since.

Turns on `move` and `stack` turn the selection about its bottom center (about
the from place for `move from … to …`); after a turned `move to <place>`, the turned
selection's lowest north-west corner lands on the place, and turned copies of
a stack line up edge to edge, however wide the turn made them.

A copy shrinks to the box around its content (blocks that are not air, block
entities and entities) and says so; `-whole` keeps the whole selection. A
paste places every block the clipboard holds, air included, so it cuts its box
into the terrain it lands in; `-skip air` keeps what is there wherever the
clipboard holds air, and `-onto air` overwrites only air. To take a build
without its water, `-drained` dries it (water becomes air, waterlogged stairs
and slabs lose their water), while `-skip water` leaves the destination as it
is where the water was:

```text
/mo load fang-manor as #fang -drained
/mo paste #fang at #site -rotate 90 -skip air
/mo paste #fang -onto air
/mo paste #fang at #site down 1 -skip air
```

Named clipboards (`#town`) are shared by all operators; without a name each
operator has their own. Copies always hold their biomes, but pastes and stacks
leave the destination's biomes alone unless they say `-with biomes`; moves
always carry them. Pastes rotate and mirror blocks and entities together;
copies get new entity identities, moved entities keep theirs. Pasted and
stacked waystones, computers and Mekanism multiblocks become new ones instead
of taking over the original's (a copied Mekanism tank forms empty and the
original keeps its contents). A cut is different: its first paste keeps the
machines' identities, so a cut and pasted computer keeps its files, and later
pastes of the same clipboard make new ones. A cut clears only blocks that are
still what it copied, so anything built meanwhile stays. Moves whose distance
is a multiple of 16 move whole sections at once and are especially fast.

## Archives

`save [#name] as town` writes a clipboard to `town.moon` in the archive
folder, `moonedit/archives` in the server directory unless the server's
`archives.folder` setting names another; `load town [as #name]` reads one back;
`archives` lists them. Archive names use letters, digits, `.`, `-` and `_`.
Both take the clipboard flags, so `save #fang as fang-dry -drained` writes a
dry copy and `load town -skip water` loads one without its water.

Saving onto a name that is taken saves nothing and offers three buttons in
chat: Overwrite, Save as the next free name (`town_1`, `town_2`, …) and
Rename…, which puts the sentence back in the chat box. `-replace` and `-next`
choose up front, which is what the console uses.

Servers on one machine that point `archives.folder` at the same absolute
folder share their archives: save a build on one, log into the other and load
it there. Archives name every block and biome, so they move between servers
and packs; archives from an older Minecraft version are upgraded when loaded,
and an archive naming blocks or biomes the server does not have is refused
rather than loaded as air.

Names ending in `.schem` are Sponge schematics, the format WorldEdit, FAWE and
most other editors read and write: `save as town.schem` exports one and
`load town.schem` imports one dropped into the archives folder (versions 1 to
3). A schematic is always a box, so blocks around a non-box selection are saved
as air; the copy point travels with it, so pastes line up as they did in the
editor that saved it. Schematics from newer Minecraft versions load with a
caution, since their block entities and entities may not fully apply; unknown
blocks and biomes are still refused.

## Brushes and the wand

`brush <shape> <edit>` binds a brush to the item in your hand. The shape is a
`sphere`, `cylinder`, `ellipsoid` or `box`, sized as in `select`; the edit is
any sentence from [Editing the selection](#editing-the-selection) or
[Shaping terrain](#shaping-terrain) except `biome`, `regen`, `count` and
`distribution`, with its flags. For example
`brush sphere 6 fill stone 70%, andesite 30% -where exposed`,
`brush sphere 5 clear -where water`, `brush box 5x3x5 drain` or
`brush cylinder 8 raise 2`.

Right-click applies the brush at the block you look at, up to 256 blocks away;
each stroke is one undoable edit. Brush shapes are centered on the block you
hit. `brush` shows the binding and `brush clear` removes it. Bound items show
their brush in their name and keep it across restarts; an item bound before a
change to the sentence language says it needs binding again. The console has
no hand: it keeps its brush in its session and applies it with
`stroke at <place>`.

`wand` turns the item in hand into a selection wand: left-click sets the first
corner, right-click the second.

A terrain brush works on the ground under its shape, even where that lies well
above or below the block you hit (16 blocks, or twice the brush's radius if
that is more, either way), and fades from full strength at its center to
nothing at its rim, so `brush sphere 8 raise 3` builds a rounded hill and
repeated strokes blend into each other; a stream instead runs at full depth
until it leaves the brush. `-soft <n>` fades over the outer n blocks instead,
and `-soft 0` gives a hard edge: `brush cylinder 10 flatten -soft 3` levels a
pad to the height you click with a short ramp around it.

Good starting points:

| Brush | For |
| --- | --- |
| `brush sphere 12 smooth 2` | tidying rough ground |
| `brush sphere 10 erode 2` | aging a slope |
| `brush sphere 6 weather` | breaking down a cliff |
| `brush sphere 16 roughen 3` | making flat land look natural |
| `brush sphere 24 mountain 40` | a peak |
| `brush sphere 20 ridge 3` | knife-edge spurs on a mountain |
| `brush cylinder 40 stream 3` | a stream to the valley, clicked high on a slope |
| `brush sphere 12 stretch -1` | turning a mound into a pond basin |

## History and jobs

`undo [n]` and `redo [n]` step through your edits; `history` lists them with
their age and the disk space history uses. History lives with the world and
survives restarts; the server's `history.diskBudgetMegabytes` setting bounds
each operator's history, and the oldest edits are dropped with a message when
it is full. Undo only reverts blocks that are still the block the edit left,
in any state (an opened door, a lit furnace, a booted computer all count), so
blocks players replaced since are kept. Containers and machines keep what they
hold through undo and redo: items delivered after an undo are still there
after the next redo and undo, and undoing a move brings back what the moved
chests hold now, not what they held when they moved.

`jobs` lists running jobs and their progress; `cancel [job]` stops one (your
newest by default), and the sentences you sent after it, still waiting, do not
run. A cancelled or failed edit keeps what it already changed, and that part
can be undone.

## The client part

moonedit also runs on the client, where it is optional: without it, everything
above works from chat. With it, an operator on a moonedit server sees their
selection, wand corners and marks outlined in the world, the shape of the brush
in their hand where a stroke would land, and, when switched on, where a paste
would go. A small corner panel shows the selection, the clipboard and running
jobs while there is something to show.

The panel opens on a key you bind under Controls, moonedit. Its sentence box
reads what is typed with the server's own grammar: it says whether the sentence
reads and where it does not, and offers what may come next (words, blocks,
biomes, your clipboards, marks and archives) as chips. Tab takes the first
chip, Enter runs the sentence, and the up and down arrows bring back earlier
ones. The palette on the left starts common sentences, grouped as Select,
Edit, Terrain, Sculpt and Clipboard; hovering an icon names it and shows the
sentence it starts. On the right are the selection, clipboards, history,
running jobs and archives, each with buttons for what applies to it. Along the
bottom, Selection, Brush, Paste and Status switch the outlines and the corner
panel on and off; the paste preview starts off. Everything the panel does is a
sentence sent to the server, behind the same permission as `/mo`.

The moonedit section of Controls lists every key the client part offers: the
panel, undo, redo and the four Selection, Brush, Paste and Status switches.
All of them start unbound, since moonedit is for operators and should not take
keys from players; bind the ones you use. Each switch names its new state above
the hotbar.

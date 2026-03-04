Game Design Doc (Living Document)
1) Elevator pitch
A queue-progression game where players begin in Hell (Layer 9) and ascend through Hell → Purgatory → Heaven, advancing by reaching the front of the line and holding it for a required time. Players earn the area’s currency based on their current queue position, and can spend it to skip the person in front. A demotion system periodically drops the last 5 queue spots down a layer, creating pressure at the back. Servers stay “full” using bots that occupy empty queue spots.

2) World structure
Areas (3): Hell, Purgatory, Heaven


Layers per area: 9 (total 27 layers)


Rule: Once you reach a new area, you can’t be demoted into the previous area.


Initial build scope: Hell only, first 3 layers (Layer 9 → 8 → 7), then expand.



3) Queues
Queue size per layer: 50 spots


Players always belong to exactly one layer/queue at a time.


Rest zone between layers.
When the line loses a player either to the front timer or 5 to the back timer the spots are replaced with bots at the end so the line always has 50 people in queue. When position 1 advances the whole line compresses so everyone will move up a spot and spot 50 will be replaced by a bot. And the queue should not change when the last 5 are removed; they will all be replaced by bots after they are removed. A bot will be added to the end every 8-15 (randomly) seconds until the queue is back to full. 
Players can not physically move around in the line they are locked in place. But they can move around in the safe areas between layers. While in queue, set WalkSpeed = 0, JumpPower = 0, and keep the character snapped to their queue node position when their spot changes (teleport them to the node (maybe later on an animation of them moving to the spot)). In rest zones, movement is restored.



4) Advancement (front hold timer)
To advance, you must reach Spot #1 (front) and hold it when the timer hits 0. The timer is always counting even if the #1 spot is constantly shifting and after reaching 0 the timer will immediately reset.


Timer resets per area.


Hell hold times: start at 30s on Hell Layer 9, increasing by +30s per layer.


H9: 30s


H8: 60s


H7: 90s


…


H1: 270s
Reaching Spot #1 and finishing hold → teleport to Rest Zone → when the player hits “Continue,” they join the next layer at the back (replacing a bot if needed).
Future plan: When new areas are added, each area introduces new threats/challenges (not just longer waits).

5) Rest zones (between layers)
The rest zone is safe and optional for up to 10 minutes.
Players can choose to continue after 30 seconds, rather than being forced to wait the full 10 minutes.


The rest zone is also a good place later for: lore, small minigames, cosmetics, “AFK safe” spots.
The rest zone will have a Demotion timer screen indicating the countdown for demotion in the next layer so players don’t accidentally join the next layer as a demotion is about to happen.



6) Demotion system (back-of-line threat)
Rule:
On a demotion tick, the game demotes the last 5 spots in the queue (spots 46–50) down one layer.


Area lock rule:
Demotion can’t send someone into a previous area.


Timer math:
 A simple starter formula for Hell:
DemotionInterval = (FrontHoldTime * 2 + 30)


H9 nothing happens as this is the first layer and you can’t be demoted from this.
H8 (60s): = 150s (2m30s)


H7 (90s): = 210s (3m30s)


H6 (120s): = 270s (4m30s)
Should there be 50 real players already in the previous layer the bottom 5 will be kicked out early rather than waiting for the time and the timer will be reset. 
Demotion Zone:
While in the demotion zone (Last 5 spots) the player will have a warning indicator on their screen with the timer until the demotion occurs. This will go away after the demotion occurs.


Although there are more people being demoted than promoted there will most likely be enough bots for it to rarely affect a player. 

7) Currency + income + skip cost (Hell formulas defined)
Currencies by area:
Hell: Sin


Purgatory: Time


Heaven: Virtue


Income is based on your queue position and increases exponentially.
Spots are numbered 1 (front) → 50 (back).


Back (spot 50) starts at 1 Sin/sec on Hell Layer 9.


Each spot closer to the front multiplies income by ×1.2.


Income formula (Hell):
Let i = your spot number (1..50), where i=50 is last.


Income(i) = BaseRate * (1.2^(50 - i))


On Hell Layer 9: BaseRate = 1 Sin/sec


Examples (Hell Layer 9):
Spot 50: 1.00 Sin/sec


Spot 47: 1.2^3 = 1.728 Sin/sec


Spot 1: 1.2^49 ≈ 7,583.7 Sin/sec (your “~9.1k” target is close enough; it still rounds up nicely)


Layer-to-layer scaling (round-up rule):
Each new layer starts at a “clean rounded” base rate at the back.


The next layer “rounds up” (ex: ~7.58k → 10k, ~75.8m → 100m).


NextLayerBaseRate = 10 ^ ceil(log10(FrontRateOfPreviousLayer))
 So H9 front (~7.6k) → H8 base = 10k.


Skip cost (core mechanic):
 Skipping the person in front costs 5 seconds of your current income, with ±10% variation.
SkipCost = Income(i) * 5 * Random(0.9, 1.1)
Should the server receive the skip from two people next to each other at the same time the server will just do nothing to the person behind (basically the skip button wont do anything but I will assume that because the timing would need to be so precise that this should essentially never happen). In case it does it will run: server validates “are you still behind the same person?” If not, deny the skip (no charge).
Example of cost (Hell Layer 9, spot 47):
Income = 1.728 Sin/sec


Base skip cost = 8.64 Sin


With ±10% variation: 7.78–9.50 Sin


Shop note:
Shops/perks exist conceptually, but are “to be added later” and not required for the core.


Other areas:
Purgatory/Heaven currency formulas are TBD later. Default plan: reuse the same structure unless decided otherwise.


Currency size handling:
Two options that can be switched in the players setting menu. Either have the numbers after 1000s be represented as 1K then 1M then 1B and so on for bigger numbers. The other option will be to handle the big numbers with scientific notation.



8) Bots (to keep queue always filled)
Each line is always 50 spots; empty spots are filled with bots.


If a player joins a layer with bots present, the player replaces a bot spot.


Early bots are easy/neutral; later bots get varied behavior/difficulty.
Bots will disappear after being “promoted’ or "demoted "; they don't actually move layers (for now at least maybe later on there will be bots that can progress).
Bots are visible npcs to other players and will have fake names and bodies so they can look like players.
Data is always full immediately (queue array instantly refills with bot records to keep 50).
Visual NPC bots spawn with delay (8–15s) so it looks natural, but the logic never has “missing spots.”
Bots are basically the same as players and can be skipped and can skip all the same.



9) Saving / persistence
Confirmed:
Save: currency, area reached, layer reached.


Do not save the exact queue position.


On rejoin, the player returns to the bottom of their saved layer, but keeps currency so they can skip back up.
On rejoin if the timer to the demotion is shorter than 15s for the layer they are going to be added to, then hold the player in the loading menu until the new demotion timer for the layer starts. For now I like the idea of having after loading in having a button that says play and if the player can’t load in it is grayed out and will turn red when the new timer starts.
Should someone leave while in a safe zone the position and layer and area will be saved as if they had progressed to the next place after the rest zone they were at. Basically they will continue at the back of the line at whatever the next layer was.

10) AFK policy
AFK is allowed; if they slide down naturally or get skipped, that’s fine.
 Roblox kicks after ~20 minutes, and since currency doesn’t decrease, they can recover when they return.

11) Leveling (to be added)

Leveling will be a small bonus that will make a player more powerful over time. The player will have access at all times to a button opening a menu for leveling upgrades. 

Players will start at lvl one and will gain xp based on the position they hold and the layer they are in. XP gain will be the same as money except the requirement for the amount of xp for each lvl is different. 

So for the xp gain it will be identical to money. 

Xp requirement will be fundamentally the same as expected. Instead of the multiplier being 1.2 it will be 1.25 per level.

Each level they get will receive 1 level point that they can spend in the leveling shop to buy 1 of 2 upgrades (for now). Multiplying their current income for money by 1+0.01 per level (so the players income at level 1 would be multiplied 1.01 and level 2 would be 1.02). The second upgrade will be the same upgrade but for XP gain instead.

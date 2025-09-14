# *REGAUGE*
## Game Summary
Hop in a mech and when yours explodes get to cover and run away from the other players that are trying to stomp you, until you get your mech drop and *REGAUGE*.

## Player
 *add diagram of structure here* <br/>
 Implementing couch co-op with Unity's input system required some structure for REGAUGE since the player needed to swap between multiple different kinds of characters. The implementation I went with is having a controller that can attach characters to itself and most of the time handle the logic for switching characters.

  <Details>
 <summary> Code </summary>

```cs

```

 </Details>

 ### Rotation
 *add diagram of structure here* <br/>
The mech rotates a lot and this requires a bit of handling since there are also a few different rotation ways.


The rotations that are being tracked by the player are two input vectors for movement and aiming, then there is the current direction of the legs and the torso which is also being smoothed. Then there is also a dash state where the player slowly becomes more responsive, the first thing that unlocks for the player after the dash is the movement rather than the ability to shoot in the aim direction. Lastly the torso starts to rotate in the input direction again. The movement direction is also used instead of the aim direction when aim input is given, the reason for this was to make the game more accessible and give the player a simpler way to interact with the game.


 <Details>
 <summary> Code </summary>

```cs

```

 </Details>

 ## Tiniest Map
 The game had a bit of a problem during a playtest which was that it was really boring to keep playing for a while. This was known ofcourse however the focus until then was to make a great base. So when we felt that we could spend some time we started adding to the breadth of the game including new maps.


<br/>*add image here* <br/>
The design behind this behemoth of a map was to be a palette cleansers between the more intricate and longer lasting maps that already existed. I wanted it to be a bunch of repeated 1v1s since all the other maps consisted of mostly large open areas where one can not completely have an uninterrupted fight.



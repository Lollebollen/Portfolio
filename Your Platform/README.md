# *Your Platform*
## Game summary
blbalbalba

## Implaementation

### Database
The largest problem with using Firebase only for asyncronus muliplayer was that there can be no server code, aside from athorety checks. This ment that the finding a level was up to the clint. which leads to a choice between storing the map data and the matchmaking data in the same place or seperate. Having them be fetched at the same time would make be simpler but would also transfer alot of uneeded data so I choose the latter. Since they are sepperate there needed to be proper checks in place so two people could not try and play against the same map at the same time. leading to me adding userID and time to the matchmaking data to check if inbetween the time you requested a map to requesting its data it had not already been retrived by another client.

<img width="546" height="254" alt="Menu" src="https://github.com/user-attachments/assets/10b8009f-8223-4361-89d8-1eea64d2d7e7" /> 

 <Details>
 <summary> Code </summary>

```cs

```

 </Details>

### level Building


#### Sellecting blocks
<img width="546" height="254" alt="Sellecting blocks" src="https://github.com/user-attachments/assets/6a5d2511-419d-426d-af60-2f79ccc70dbe" />

#### Placing blocks
<img width="546" height="254" alt="Sellecting blocks" src="https://github.com/user-attachments/assets/58b67f38-4d8a-4550-8c17-8cc8a1a9a170" />

### Replays
add image here

The replays just initilize the level like usual but instead of a player it just spawns a ghost that interpulates between a bunch of points.
 <Details>
 <summary> Code </summary>

```cs

```

 </Details>

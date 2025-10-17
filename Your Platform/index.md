# *Your Platform*

### [Main page](..) 
---

## Game summary
*Your Platform* is an asynchronous multiplayer 2D platformer, where you place down blocks to aid yourself or make the next person fail.  

## Implementation

### Database
The largest problem with using Firebase only for asynchronous multiplayer was that there can be no server code, aside from authority checks. This meant that the finding a level was up to the client. Which leads to a choice between storing the map data and the matchmaking data in the same place or separately. Having them be fetched at the same time would make it simpler but would also transfer a lot of unneeded data so I choose the latter. Since they are separate there needed to be proper checks in place so two people could not try and play against the same map at the same time. leading to me adding userID and time to the matchmaking data to check if in between the time you requested a map to request its data it had not already been retrieved by another client.

<img width="546" height="254" alt="Menu" src="https://github.com/user-attachments/assets/10b8009f-8223-4361-89d8-1eea64d2d7e7" /> 

 <Details>
 <summary> Code </summary>

<pre>
<code>

private void CheckForOpenGame(DataSnapshot snapShot)
{
    GameStates gameStates = JsonUtility.FromJson&lt;GameStates&gt;(snapShot.GetRawJsonValue());

    for (int i = 0; i &lt; gameStates.activeStatus.Length; i++)
    {
        long time = gameStates.activeStatus[i].timeSetActive;
        if (!gameStates.activeStatus[i].isActive || (time &lt; System.DateTime.UtcNow.Ticks && new System.DateTime(time).Day != System.DateTime.UtcNow.Day))
        {
            SetGameActive(i);
            FetchGameData(i);
            return;
        }
    }

    startButton.interactable = true;
    Debug.LogWarning("No game is active, Try again later");
}

private void SetGameActive(int i)
{
    Active active = new Active(true, System.DateTime.UtcNow.Ticks);
    database.RootReference.Child("gameStates").Child("activeStatus").Child(i.ToString()).SetRawJsonValueAsync(JsonUtility.ToJson(active)).ContinueWithOnMainThread(task =&gt;
    {
        if (task.Exception != null) { Debug.Log(task.Exception); }
        else
        {
            gameStateindex = i;
            startTime = active.timeSetActive;
            TryToStartGame(false);
        }
    });
}

private void FetchGameData(int i)
{
    database.RootReference.Child("games").Child("gameData").Child(i.ToString()).GetValueAsync().ContinueWithOnMainThread(task =&gt;
    {
        if (task.Exception != null) { Debug.Log(task.Exception); }
        else
        {
            gameDataindex = i;
            gameData = task.Result.GetRawJsonValue();
            TryToStartGame(false);
        }
    });
}

private void TryToStartGame(bool isReplay)
{
    if (gameDataindex == gameStateindex && gameData != null && gameData.Length &gt; 1 && startTime != default)
    {
        StartGame(isReplay);
    }
}

</code>
</pre>
</Details>

## level Building


#### Selecting blocks

The building is the main part of the game and it could not be to fomrulaic or boring. The solution I went with was to make it random and make the selection take two stages. The random is weighted to make it more inpactful to get the more rare types of blocks. While making it a two step process add more structure and skill into the mechaniq, aswell as hopefully making the random aspect more engaging the hopeing for something good.

<img width="546" height="254" alt="Sellecting blocks" src="https://github.com/user-attachments/assets/6a5d2511-419d-426d-af60-2f79ccc70dbe" />

#### Placing blocks

The placing is heavily inspierd by _Ballons Tower Defens_ and _Plants vs Zombies 2_, since they both let you drag objects into a playarea with your fingers. My main gripe with thier implementation is the inacurase of not being able to see exactly were you are placing down. The way I got around this is using the fact that my building takes place in a stress free enviroment were an extra input will not stress the player, specificly I made it so you can drag from anywhere on the screen and then you just need to accept the position.

<img width="546" height="254" alt="Sellecting blocks" src="https://github.com/user-attachments/assets/58b67f38-4d8a-4550-8c17-8cc8a1a9a170" />

### Replays

<img width="546" height="254" alt="Replay" src="https://github.com/user-attachments/assets/efe81cf7-b1b3-4130-9544-149d0498ccae" />

The replays just initilize the level like usual but instead of a player it just spawns a ghost that interpulates between a bunch of points.
 <Details>
 <summary> Code </summary>

<pre>
<code>

using UnityEngine;

public class GhostPlayer : MonoBehaviour
{
    Ghost ghost;
    float startTime;
    int i = 1;

    public void SetGhost(Ghost ghost)
    {
        this.ghost = ghost;
        startTime = Time.time;
    }

    private void Update()
    {
        if (ghost == null) { return; }
        FollowGhost();
    }

    private void FollowGhost()
    {
        float ratio;
        do
        {
            if (Time.time - startTime &gt; ghost.times[ghost.times.Length - 1]) { Finished(); return; }
            ratio = (Time.time - startTime - ghost.times[i - 1]) / (ghost.times[i] - ghost.times[i - 1]);
            if (ratio &lt; 1) { break; }
            else { i++; }
        } while (true);

        Vector3 newPos = new Vector3(ratio * (ghost.x[i] - ghost.x[i - 1]) + ghost.x[i - 1], ratio * (ghost.y[i] - ghost.y[i - 1]) + ghost.y[i - 1], transform.position.z);
        transform.position = newPos;
    }

    private void Finished()
    {
        LevelManager.Instance.OnReplayComplete();
    }
}

</code>
</pre>
</Details>
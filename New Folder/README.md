# *New Folder*
## Game Summary
blalballa

## Creating a File System 
#### Saving 
 At first when trying to save a windos scritable object to an icon I used InstanceID, it worked until it didn't. Since it only referce to the same object during the same runtime, which meant it could not be used in such a way. After finding out that the REF folder should idealy not be used I seateld on using a refrence list of all of the window datas instead.

 Saving to JSON only needed to be done during development which ment it could be stored inside a scritable object

 <Details>
 <summary> Code </summary>

```cs

```

 </Details>


 #### Data Structure
The data structure is just folders holding files and its own folder heiarcy. Each folder has a GUID that can be used to retrive it's data and the desktop is just a uneqie folder that opens up automaticly. The folder heiarcy is stored inside the folder to make the check to see if you are trying to place a folder insde itself easier.

 <Details>
 <summary> Code </summary>

```cs

```

 </Details>

#### Folder Action
add image here

 <Details>
 <summary> Scatter code </summary>

```cs

```

 </Details>



 
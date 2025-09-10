# *New Folder*
## Game Summary
*New Folder* is a horror puzzle game where you dive into an old pc and figure out passwords to discover mysteries.

## Creating a File System 
 #### Data Structure
The data structure is just folders holding files and its own folder hierarchy. Each folder has a GUID that can be used to retrieve its data and the desktop is just a unique folder that opens up automatically. The folder hierarchy is stored inside the folder to make the check to see if you are trying to place a folder inside itself easier.

<img width="546" height="254" alt="Data structure" src="https://github.com/user-attachments/assets/9505b8a3-44bc-4434-a87a-0e7fc20b580e" />

 <Details>
 <summary> Code </summary>

```cs

```

 </Details>

#### Saving 
 At first when trying to save a windows scriptable object to an icon I used InstanceID, it worked until it didn't. Since it only references the same object during the same runtime, which means it could not be used in such a way. After finding out that the resource folder should ideally not be used, I settled on using a reference list of all of the windows data instead.


 Saving to JSON only needed to be done during development which meant it could be stored inside a scriptable object.


 <Details>
 <summary> Code </summary>

```cs

```

 </Details>

#### Folder Action
To make the game feel as immersive as possible it needed to feel like a computer which made me add as many small quirks of file managemt that time would allow. Some of these small things that are not necisary for the gameplay include Multiselecting, renaming files and shortcuts. 



add image here

 <Details>
 <summary> Scatter code </summary>

```cs

```

 </Details>




 

# *New Folder*
### Game Summary
*New Folder* is a horror puzzle game where you dive into an old pc and figure out passwords to discover mysteries.

## Creating a File System 
 #### Data Structure
The data structure is just folders holding files and its own folder hierarchy. Each folder has a GUID that can be used to retrieve its data and the desktop is just a unique folder that opens up automatically. The folder hierarchy is stored inside the folder to make the check to see if you are trying to place a folder inside itself easier.

<img width="546" height="254" alt="Data structure" src="https://github.com/user-attachments/assets/9505b8a3-44bc-4434-a87a-0e7fc20b580e" />

 <Details>
 <summary> Code </summary>
Irrelevant code removed

```cs
using System;
using UnityEngine;

[Serializable]
public class FolderData
{
    public bool[] isLockeds;
    public bool[] canBeDestroyeds;
    public string[] passwords;
    public string[] iconNames;
    public string[] folderGuids;
    public string[] folderHierarchy;
    public string[] windowDatas;
    public string[] pinWindowDatas;
    public int[] xCoordinates;
    public int[] yCoordinates;
    public GameObject[] folderWindows;
    public Icon.IconType[] iconTypes;
}
```

```cs
using UnityEngine;

public class FolderHolder : MonoBehaviour
{
    public static FolderHolder instance;
    public SerializableDictionary<string, FolderData> folderDatas = new();
    uint count = 0;

    public string GenerateFolderGUID()
    {
        count++;
        Hash128 hash = new();
        hash.Append(Time.time);
        hash.Append(count);
        string GUID = hash.ToString();
        folderDatas.Add(GUID, new());
        return GUID;
    }
}
```

```c

using UnityEngine;

namespace FolderDataFunctions
{
    public class DataFunctions
    {
        public static void InitializeFolderDataArrays(FolderData folderData, int num)
        {
            folderData.isLockeds = new bool[num];
            folderData.canBeDestroyeds = new bool[num];
            folderData.folderWindows = new GameObject[num];
            folderData.passwords = new string[num];
            folderData.iconNames = new string[num];
            folderData.xCoordinates = new int[num];
            folderData.yCoordinates = new int[num];
            folderData.folderGuids = new string[num];
            folderData.iconTypes = new Icon.IconType[num];
            folderData.windowDatas = new string[num];
            folderData.pinWindowDatas = new string[num];
        }

        public static void DefineFolderDataArrays(FolderData folderData, Icon icon, int i)
        {
            if (icon.folderWindow != null)
            {
                folderData.folderWindows[i] = icon.folderWindow.gameObject;
            }
            folderData.canBeDestroyeds[i] = icon.canBeDestroyed;
            folderData.iconNames[i] = icon.iconName;
            folderData.xCoordinates[i] = icon.gridCoordinates.x;
            folderData.yCoordinates[i] = icon.gridCoordinates.y;
            folderData.windowDatas[i] = FolderHolder.instance.GenerateWindowGUID(icon.windowData);
            folderData.folderGuids[i] = icon.iconGUID;
            folderData.iconTypes[i] = icon.iconType;
            folderData.isLockeds[i] = icon.isLocked;
            folderData.passwords[i] = icon.hasGlitched;

            if (icon.pinWindowData != null)
            {
                folderData.pinWindowDatas[i] = FolderHolder.instance.GenerateWindowGUID(icon.pinWindowData);
            }
        }

        public static void DefineFolderDataArrays(FolderData folderData, FolderData oldFolderData, int i)
        {
            folderData.canBeDestroyeds[i] = oldFolderData.canBeDestroyeds[i];
            folderData.folderWindows[i] = oldFolderData.folderWindows[i];
            folderData.iconNames[i] = oldFolderData.iconNames[i];
            folderData.xCoordinates[i] = oldFolderData.xCoordinates[i];
            folderData.yCoordinates[i] = oldFolderData.yCoordinates[i];
            folderData.windowDatas[i] = oldFolderData.windowDatas[i];
            folderData.folderGuids[i] = oldFolderData.folderGuids[i];
            folderData.iconTypes[i] = oldFolderData.iconTypes[i];

            folderData.isLockeds[i] = oldFolderData.isLockeds[i];
            folderData.passwords[i] = oldFolderData.passwords[i];
            folderData.pinWindowDatas[i] = oldFolderData.pinWindowDatas[i];
        }

        public static Icon DefineIcon(FolderData folderData, GameObject iconGameObject, int i)
        {
            Icon icon = iconGameObject.GetComponent<Icon>();
            Vector2Int coordinates = new(folderData.xCoordinates[i], folderData.yCoordinates[i]);
            icon.canBeDestroyed = folderData.canBeDestroyeds[i];
            icon.gridCoordinates = coordinates;
            icon.folderIsActive = folderData.folderWindows != null && folderData.folderWindows[i] != null;
            icon.iconName = folderData.iconNames[i];
            icon.windowData = FolderHolder.instance.GUIDToWindowData(folderData.windowDatas[i]);
            icon.iconGUID = folderData.folderGuids[i];
            icon.iconType = folderData.iconTypes[i];
            icon.folderHierarchy = folderData.folderHierarchy;
            icon.isLocked = folderData.isLockeds[i];
            icon.hasGlitched = folderData.passwords[i];

            if (folderData.pinWindowDatas[i] != null && folderData.pinWindowDatas[i] != "")
            {
                LockScript lockScript = icon.AddComponent<LockScript>();
                icon.pinWindowData = FolderHolder.instance.GUIDToWindowData(folderData.pinWindowDatas[i]);
                icon.pinWindowData.SetPinWindowData(lockScript);
            }

            return icon;
        }
    }
}
```

 </Details>

#### Saving 
 At first when trying to save a windows scriptable object to an icon I used InstanceID, it worked until it didn't. Since it only references the same object during the same runtime, which means it could not be used in such a way. After finding out that the resource folder should ideally not be used, I settled on using a reference list of all of the windows data instead.


 Saving to JSON only needed to be done during development which meant it could be stored inside a scriptable object.


 <Details>
 <summary> Code </summary>

```cs

using UnityEngine;

[CreateAssetMenu(fileName = "New Save Data", menuName = "Save Data")]
public class SaveData : ScriptableObject
{
    public string desktopData;
    public string desktopDataStorage;
    public string newDesktopData;
}
 

```

```cs
using System.Collections.Generic;
using UnityEngine;

public class FolderHolder : MonoBehaviour
{
    public static FolderHolder instance;
    public SerializableDictionary<string, FolderData> folderDatas = new();
    public SaveData saveData;
    [SerializeField] WindowData[] windowDatas;

    Dictionary<string, WindowData> windowDataLookUpTable = new Dictionary<string, WindowData>();


    private void Awake()
    {
        var folderHolders = FindObjectsOfType<FolderHolder>();
        if (folderHolders.Length == 1)
        {
            instance = this;
        }
        else
        {
            Destroy(gameObject);
        }

        if (saveData.desktopData != "")
        {
            Load();
        }

        PopulateWindowDataLookUpTable();
    }

    public void Load()
    {
        folderDatas = JsonUtility.FromJson<SerializableDictionary<string, FolderData>>(saveData.desktopData);
    }

    private void PopulateWindowDataLookUpTable()
    {
        foreach (WindowData windowData in windowDatas)
        {
            windowDataLookUpTable.Add(GenerateWindowGUID(windowData), windowData);
        }
    }

    public string GenerateWindowGUID(WindowData windowData)
    {
        Hash128 hash = new();
        hash.Append(windowData.name);
        string GUID = hash.ToString();
        return GUID;
    }

    public WindowData GUIDToWindowData(string GUID)
    {
        if (!windowDataLookUpTable.ContainsKey(GUID))
        {
            Debug.LogWarning("The window data needs to be inside Game handler to be used. GUID: " + GUID);
            return null;
        }
        return windowDataLookUpTable[GUID];
    }
}
```

 </Details>

#### Folder Action
To make the game feel as immersive as possible it needed to feel like a computer which made me add as many small quirks of file management that time would allow. Some of these small things that are not necessary for the gameplay include multiselect, renaming files and shortcuts.


![FolderAction](https://github.com/user-attachments/assets/bfaf8492-ebe3-4942-b039-6930182ac8f3)

 <Details>
 <summary> Scatter code </summary>

```cs
    public string[] GetSubfolders(string originFolder, out Dictionary<string, string> lockedFolders)
    {
        List<string> subfolderers = new();
        Stack<string> folderStack = new();
        lockedFolders = new();
        subfolderers.Add(originFolder);

        int j = 0;
        foreach (string folderGUID in folderDatas[originFolder].folderGuids)
        {
            if (string.IsNullOrEmpty(folderGUID))
            {
                j++;
                continue;
            }
            if (folderDatas[originFolder].isLockeds[j])
            {
                j++;
                lockedFolders.Add(folderGUID, folderGUID);
                continue;
            }
            subfolderers.Add(folderGUID);
            folderStack.Push(folderGUID);
            j++;
        }

        int i = 0;
        while (folderStack.Count > 0 && i < 100)
        {
            int k = 0;
            FolderData folderData = folderDatas[folderStack.Pop()];
            if (folderData.folderGuids != null)
            {
                foreach (string folderGUID in folderData.folderGuids)
                {
                    if (string.IsNullOrEmpty(folderGUID))
                    {
                        k++;
                        continue;
                    }
                    if (folderData.isLockeds[k])
                    {
                        k++;
                        lockedFolders.Add(folderGUID, folderGUID);
                        continue;
                    }
                    subfolderers.Add(folderGUID);
                    folderStack.Push(folderGUID);
                    k++;
                }
                i++;
            }
        }
        return subfolderers.ToArray();
    }

    public void ScatterIcons(GlitchEvents.ScatterType scatterType, string folderOpened, GameObject iconBase, float totalScatterTime, Icon triggerIcon)
{
    SelectManager.instance.ClearSelected();
    string originFolder;
    string triggerIconGUID = null;
    IconGrid triggerIconFolder = null;
    FolderData triggerFolderData = null;
    if (triggerIcon != null)
    {
        triggerIconGUID = triggerIcon.iconGUID;
        triggerFolderData = FolderHolder.instance.folderDatas[triggerIconGUID];
        triggerIconFolder = triggerIcon.folderWindow;
    }

    switch (scatterType)
    {
        case GlitchEvents.ScatterType.DESKTOP:
            originFolder = IconGrid.desktopFalseGUID;
            break;
        case GlitchEvents.ScatterType.TRASHCAN:
            Debug.LogWarning("This scatter type was depricated: implemnt");
            originFolder = IconGrid.trashcanFalseGUID;
            break;
        case GlitchEvents.ScatterType.OPENED:
            Debug.LogWarning("This scatter type was depricated: implemnt");
            if (!FolderHolder.instance.folderDatas.ContainsKey(folderOpened)) { return; }
            originFolder = folderOpened;
            break;
        default:
            return;
    }

    IconGrid[] iconGrids = FindObjectsOfType<IconGrid>();
    IconGrid desktop = GetComponent<IconGrid>();
    foreach (IconGrid iconGrid in iconGrids)
    {
        if (triggerIcon != null && triggerIcon.folderWindow == iconGrid) { continue; }
        if (iconGrid.thisFolderGUID != IconGrid.desktopFalseGUID)
        {
            iconGrid.RemoveAllIcons();
            continue;
        }
        desktop = iconGrid;
        if (scatterType == GlitchEvents.ScatterType.DESKTOP)
        {
            iconGrid.RemoveAllIcons();
        }
    }

    CursorLockMode pastLockState = Cursor.lockState;
    Cursor.lockState = CursorLockMode.Locked;
    Cursor.visible = false;

    StartCoroutine(ScatterIconsOneByOne(scatterType, iconBase, originFolder, desktop, pastLockState, totalScatterTime, triggerIcon,
        triggerFolderData, triggerIconGUID, triggerIconFolder));
}

IEnumerator ScatterIconsOneByOne(GlitchEvents.ScatterType scatterType, GameObject iconBase, string originFolder, IconGrid desktop,
    CursorLockMode pastLockState, float totalScatterTime, Icon triggerIcon, FolderData triggerFolderData, string triggerIconGUID, IconGrid triggerIconFolder)
{
    Icon newOriginIcon = null;
    List<Icon> icons = new List<Icon>();
    Vector3 topRight = Camera.main.ViewportToWorldPoint(new Vector3(1, 1, 0));
    Vector3 bottomLeft = Camera.main.ViewportToWorldPoint(new Vector3(-1, -1, 0)) / 2;
    string[] folders = FolderHolder.instance.GetSubfolders(originFolder, out Dictionary<string, string> lockedFolders);
    int iconNum = 0;
    foreach (string folder in folders)
    {
        if (FolderHolder.instance.folderDatas[folder].iconNames == null) { continue; }
        iconNum += FolderHolder.instance.folderDatas[folder].iconNames.Length;
    }
    float scatterTime = 0;
    if (totalScatterTime > 0) { scatterTime = totalScatterTime / iconNum; }

    foreach (string folder in folders)
    {
        FolderData folderData = FolderHolder.instance.folderDatas[folder];
        if (folderData == null || folderData.iconNames == null) { continue; }

        for (int i = 0; i < folderData.iconNames.Length; i++)
        {
            if (folderData.iconTypes[i] == Icon.IconType.Trashcan && folder == triggerIconGUID) { continue; }
            yield return new WaitForSeconds(scatterTime);
            GameObject iconGameobject = Instantiate(iconBase);
            Icon icon = DataFunctions.DefineIcon(folderData, iconGameobject, i);
            icon.ChangeIconGrid(desktop);
            if (scatterType == GlitchEvents.ScatterType.OPENED && folder == originFolder)
            {
                Icon toRemoveIcon = triggerIcon.folderWindow.icons[new Vector2Int(folderData.xCoordinates[i], folderData.yCoordinates[i])];
                triggerIcon.folderWindow.RemoveIcon(toRemoveIcon.gridCoordinates);
                Destroy(toRemoveIcon.gameObject);
            }

            Vector3 randomPos = new Vector3(UnityEngine.Random.Range(bottomLeft.x, topRight.x), UnityEngine.Random.Range(bottomLeft.y, topRight.y), 0);
            desktop.AddIcon(icon, randomPos, true, false);
            icons.Add(icon);

            if (icon.iconGUID == triggerIconGUID)
            {
                newOriginIcon = icon;
            }
        }

        if (!lockedFolders.ContainsKey(folder) && folder != desktop.thisFolderGUID)
        {
            FolderData newFolderData = new();
            DataFunctions.InitializeFolderDataArrays(newFolderData, 0);
            desktop.SaveFolderData(newFolderData, folder);
        }
    }

```

 </Details>

## Windows and Unity UI

#### Learning Unitys UI
Simply put, this game revolves around ui, so therefore I needed to know how to use it. There was a lot of fighting with the ui since we used world space canvas and Unity's UI components were not made for it. As an example Unity's layout groups use ints which works fine when the measurements are in pixels but when they are in unity units you lose a lot of fine control over the visuals. This specifically was counteracted by using two Gameobjects one with the layout group and the other one that was childed and used a stretch anchor to get proper control over the visuals.

#### Window data

The window data scriptable objects was made so other people could expand on it. However it did miss on this since along the development it was needed to have one data open two different kinds of windows to sell the glitch effect. So the solution was to simply have two methods, one standard and another for exceptions.

 <Details>
 <summary> Code </summary>

```cs
using UnityEngine;

public class WindowData : ScriptableObject
{
    public string fileEnding;

    public void SetWindowData(GameObject window, Icon icon)
    {
        SettingWindowData(window, icon);
        SaveWindowDataOnWindow(window);
    }

    public void SetWindowData2(GameObject window, Icon icon)
    {
        SettingWindowData2(window, icon);
        SaveWindowDataOnWindow(window);
    }

    public virtual void SettingWindowData(GameObject window, Icon icon)
    {
        Debug.LogWarning("Base window data was used");
    }

    public virtual void SettingWindowData2(GameObject window, Icon icon)
    {
        Debug.LogWarning("Base window data was used");
    }

    public virtual void SetPinWindowData(LockScript lockScript)
    {
        Debug.LogWarning("Non lockscipt window data is used as lockscript window data", lockScript);
    }

    private void SaveWindowDataOnWindow(GameObject window)
    {
        window.GetComponent<CloseWindow>().windowData = this;
    }
}

```

 </Details>

### Tools
In retrospect there needed to be tools built alongside my implementations of a lot of these systems. Since other people were able to set up a scene and make window datas fine, the process however was needlessly tedious and clunky. Even simple things could have helped such as being able to select the current folder or icon in the hierarchy if you had it selected in game.



 


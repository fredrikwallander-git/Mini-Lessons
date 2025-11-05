# Saving and Loading in Unity

Often we want to save or load the current state of progression in our games so that we don't have to start from scratch every single time we start up and enter a game.

Whether it is coins, the current level, upgrades and various other kinds of progression data - it all has to be stored somewhere and somehow!

The most common ways to store data is through:
- The Unity `PlayerPrefs` class
- A file stored on the device
- Or cached on a server

Each one offers their own pros and cons.. which one you chose depends on your needs!

---
## PlayerPrefs

The PlayerPrefs class is a simple, platform-independent way to save and load small amounts of data between game sessions.

It can store:
- `ints`
- `floats`
- `strings`

Each piece of data is accessed using a key (a text string) to save, load or even delete the stored data.

The data is stored in a local registry on the device that varies by platform such as:
- the Windows **registry**
- a `.plist` file on macOS / iOS
- or a **shared preferences** file for android

Although a straightforward and nice way of storing data, it is only intended for small non-complex data and offers no encryption and can be easily accessed by the user.

Limitations:
- **data type restrictions**.. string, float and integer types
- **no encryption**, data is stored as plain text
- frequent read and writes can impact performance, especially on mobile devices

---

Let's take a look at how to use the `PlayerPrefs`:

```csharp
// Save data
PlayerPrefs.SetInt("HighScore", 1500);
PlayerPrefs.Save();

// Load data
int highScore = PlayerPrefs.GetInt("HighScore", 0);
Debug.Log("High Score: " + highScore);

// Optional check for key
if(PlayerPrefs.HasKey("HighScore") 
{
    Debug.Log("The key: <HighScore> exists.");
}
else
{
    Debug.Log("The key: <HighScore> does not exists.");
}
```

In this example we save the high score value of **1500** to the player prefs with the `key` string of `"HighScore"`.

Afterward we can use the same `key` to load the data, if there is any, or load `0` if there has not been any data saved yet to the specific `key`.

Optionally: You can check if the key exists prior to getting the data, but you do not have to since PlayerPrefs is forgiving and returns a default value or a specified default value.

---

## JSON (JavaScript Object Notation)

Another way to store data is through the combination of a locally saved file and data converted into the human-readable `JSON` format.

It works with items that are marked as `Serializable` and can be converted into a formatted strin which can then be saved to the a file on the disk.

Note that again, the data can easily be manipulated and offers no encryption or safety to prevent cheating.

Here's an example of a serialized class with data in JSON format:
```json
{
  "Coins" : 15,
  "Level" : "CandyLevel",
  "Gems" : 999
}
```

Here we have an object which contains the fields: `Coins`, `Level` and `Gems` and their stored values.


## Example code

```csharp
using UnityEngine;
using System.IO;

[System.Serializable]
public class PlayerData
{
    public int coins;
    public int lives;
    public string currentLevel;
}

public class SaveManager : MonoBehaviour
{
    string savePath;

    void Awake()
    {
        savePath = Application.persistentDataPath + "/save.json";
    }

    public void SaveGame(PlayerData data)
    {
        string json = JsonUtility.ToJson(data);
        File.WriteAllText(savePath, json);
    }

    public PlayerData LoadGame()
    {
        if (File.Exists(savePath))
        {
            string json = File.ReadAllText(savePath);
            return JsonUtility.FromJson<PlayerData>(json);
        }
        return new PlayerData(); // default
    }
}
```

Here we have created a PlayerData class which can be serialized and stored as a JSON text into a file of our chosing.

We have a manager that handles saving and loading the data that points to the specific save path of our save file.

---

## Best way?

Though both of these two offers a great way to store both small, non-complex data or larger more complex data, neither of them have any way of preventing changing the data outside the game.

In order for us to safely store and load data is through the means of a backend implementation with cloud storage so that we can handle safety checks and decide what is right or wrong.

But this is for another chapter and for our small games we make now, we need not worry about cheat prevention and safety :)
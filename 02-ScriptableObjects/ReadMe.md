# Scriptable Objects

## Concept 

Scriptable objects are reusable and editable data containers that situates as assets inside your project.

They are perfect for things like:
- item definitions
- player / enemy stats
- level settings
- audio profiles
- .. and much much more

They make it easy to tweak the data inside the inspector and reuse it across prefabs and scenes.

---

### Example: Power-up definition

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "NewPowerUp", menuName = "Game/PowerUp")]
public class PowerUpData : ScriptableObject
{
    public string powerUpName;
    public Sprite icon;
    public float duration;
    public float speedMultiplier;
}
```

Now you can create PowerUp assets from the menu:

>Assets → Create → Game → PowerUp

Then in your PowerUp MonoBehaviour:

```csharp
using UnityEngine;

public class PowerUp : MonoBehaviour
{
    public PowerUpData data;

    void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            StartCoroutine(ApplyPowerUp(other.GetComponent<PlayerController>()));
        }
    }

    IEnumerator ApplyPowerUp(PlayerController player)
    {
        player.speed *= data.speedMultiplier;
        yield return new WaitForSeconds(data.duration);
        player.speed /= data.speedMultiplier;
    }
}
```

---

### Example: Player Settings

Define a scriptable object of `PlayerSettings` and add any meaningful values.

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "NewPlayerSettings", menuName = "Game/PlayerSettings")]
public class PlayerSettings : ScriptableObject
{
    public float movementSpeed;
    public float speedMultiplier;
    public float jumpHeight;
}
```

Then in our player class we can use this as a single field instead of having all the settings inside our player class.

```csharp
using UnityEngine;

public class Player : MonoBehaviour
{
    public PlayerSettings playerSettings;
    
    // use for movement, health, speed etc..
}
```

---

### Data persistence

When using a `ScriptableObject` it is important to note that **ANY** changes in the **inspector** of it, in or outside of play mode, affects the `ScriptableObject` asset.

Previously you might have noticed that if we change something in the **inspector** of a `GameObject` it is reverted when we exit play mode, so it might be confusing when you start to use a `ScriptableObject` and they retain the changes.

---
### **Important Note**

Unity doesn’t automatically save changes to a `ScriptableObject` made via script in Edit mode. In these cases, you must call `EditorUtility.SetDirty();` on the ScriptableObject to ensure Unity’s serialization system recognizes it as changed and saves the changes to disk. Without this, changes may not persist between Editor sessions.

Without the call to `EditorUtility.SetDirty();`, the change appears in memory, but if you close and reopen the Editor the value reverts to its previous value.

Another option is to use AssetDatabase.SaveAssets(); to save immediately if you really need the save to happen instantly and not when Unity decides to do it.

---
### Messages

As with a `MonoBehaviour`, the `ScriptableObject` also comes with some events that are triggered at certain points.
Use it to your advantage when you see a need for it.

They are:
- `Awake()` -> Called when a scriptable object is created
- `OnDestroy()`	-> Called when the scriptable object will be destroyed.
- `OnDisable()`	-> Called when the scriptable object goes out of scope.
- `OnEnable()` -> Called when the object is loaded.
- `OnValidate()` -> Editor-only function that Unity calls when the script is loaded or a value changes in the Inspector.
- `Reset()` -> Resets the scriptable object to default values.


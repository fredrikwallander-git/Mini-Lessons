# Type Of T - Generics Expanded

In game development (and software development in general), we often run into situations where we want to write logic that works with **different kinds of data, objects or components** - without rewriting the same method or class for each type.

For example:
- A `player` can pick up **coins**, **keys**, **potions**, **weapons**..
- An `Enemy` could be a **slime**, **bat**, **robot**, **boss**..
- A `Stat` could be **health**, **speed**, **mana**, **strength**..

We _could_ write separate classes/methods like:

```csharp
void AddCoin(Coin coin); { ... }
void AddKey(Key key); { ... }
void AddPotion(Potion potion); { ... }
```

But that's:
- Repetitive
- Hard to maintain
- Hard to expand

### Cue Generics - to the rescue!

```csharp
void Add<T>(T item)
{
    // works for ANY type T
}
```

`T` is a placeholder for a type.

When you use the method, `T` becomes the actual type you pass into it.

You've already seen generics, maybe without even realizing it:

| Code                 | T Represents                                 |
|----------------------|----------------------------------------------|
| GetComponent<T>()    | The component type you want                  |
| List<T>              | A list of ANY type (Item, Vector3, Enemy...) |
| Dictionary<K,V>      | A mapping between two types (Key and Value)  |
| TryGetComponent<T>() | Try to retrieve a component of type T        |

When you write:

```csharp
GetComponent<RigidBody2D>();
```

You are actually calling:

```csharp
GetComponent<T>(); // Where T = RigidBody2D
```

---

## Example: Inventory using List<T>

Instead of saying the inventory can only hold **Coins**, we allow it to store **any Item type**.

```csharp
public class PlayerInventory : MonoBehaviour 
{
    public List<Item> items = new List<Item>();
    
    public void AddItem<T>(T item) where T : Item
    {
        items.Add(item);
        Debug.Log("Added: " + item.name);
    }
}
```

Usage:

```csharp
AddItem(potion);
AddItem(weapon);
AddItem(key);
```

`T` = **_Potion_**, **_Weapon_**, **_Key_**, depending on what you pass in.

## In Classes

Let's take a look at a class example:

```csharp
public class Stat<T> 
{
    public T Value;
    
    public Stat(T value)
    {
        Value = value;
    }
}
```

Usage:
```csharp
Stat<int> health = new Stat<int>(100);
Stat<float> speed = new Stat<float>(5.5f);
Stat<float> critChance = new Stat<float>(0.25f);
```

### Deeper understanding of `ref`

Normally when you pass a variable to a method, the method receives a **copy**, not the original.

`ref` allows the method to modify the **_original_** variable.

Take a look at this example:

```csharp
void Heal(ref int health, int amount)
{
    health += amount; // adjust the value of the actual variable and not a copy of the original
}
```

Usage:

```csharp
int playerHealth = 50;
Heal(ref playerHealth, 10);
```

Think of `ref` as **giving the method direct access to the real health stat**, not a clone of it.

## Generic Constraints (where T: ...)

We already know that:

```csharp
List<T> // T = type stored in the list
GetComponent<T>() // T = type of component
TryGetComponent<T>(out T item) // ... etc
```
`T` is a _placeholder_ _type_ used when writing reusable code.

And sometimes we want to **_restrict_** what `T` can be.

This is where generic constraints comes in handy:

| Syntax                  | Meaning                                         | When to use example                        |
|-------------------------|-------------------------------------------------|--------------------------------------------|
|` where T : class         `| T _**must**_ be a reference of type                   | To require object behavior ( not numbers ) |
|` where T : struct        `| T _**must**_ be a value type (int, float, Vector3...) | For math, stat structs, data-only types    |
|` where T : MonoBehaviour `| T _**must**_ be a Unity component                     | For poolable scene objects                 |
|` where T : Item          `| T _**must**_ inherit from your base class of Item     | For inventories / item systems             |
|` where T : new()         `| T _**must**_ have an empty constructor                | For data containers created in code        |

### where T : MonoBehaviour

Used when your generic type _must_ be something attached to a GameObject.

Example:
```csharp
public class HealthBar<T> where T : Monobehaviour
{
    public T uiComponent;
}
```

This prevents someone accidentally using:
```csharp
HealthBar<int> ( int is not a monobehaviour type )
```

### where T : struct

Used when `T` _must_ be a **value type**, such as:

- `int`
- `float`
- `bool`
- `Vector2`, `Vector3`

Example: A generic stat that only works with numbers.
```csharp
public class Stat<T> where T : struct
{
    public T Value;
    public Stat(T value) { Value = value; }
}
```

### where T : new()

This requires `T` to have a **parameterless constructor**.
```csharp
public class DataWrapper<T> where T : new()
{
    public T Data = new T();
}
```

This doesn't work for `MonoBehaviours`, since they cannot be constructed via `new()`.

## Object pooling with generics

Why pooling?
- No Instantiate() overhead during gameplay
- No Destroy() memory spikes
- Great for:
  - Bullets
  - NPCs that spawn / despawn
  - Explosion effects
  - Collectible
  - Cars
  - ...

### Generic Object Pool

```csharp
using System.Collections.Generic;
using UnityEngine;

public class ObjectPool<T> where T : MonoBehaviour
{
    private Queue<T> pool = new Queue<T>();
    private T prefab;
    private Transform poolParent;
    
    public ObjectPool<T>(T prefab, int initialSize, Transform poolParent)
    {
        this.prefab = prefab;
        this.poolParent = poolParent;
        for(int i = 0; i < initialSize; i++)
            CreateNewObject();
    }
    
    private void CreateNewObject()
    {
        T newObj = GameObject.Instantiate(prefab, poolParent);
        newObj.gameObject.SetActive(false);
        pool.Enqueue(newObj);
    }
    
    public T Get(Transform parent = null)
    {
        if(pool.Count == 0)
            CreateNewObject();
        
        T obj = pool.Dequeue();
        obj.transform.SetParent(parent);
        obj.gameObject.SetActive(true);
        return obj;
    }
    
    public void Return(T obj)
    {
        obj.gameObject.SetActive(false);
        obj.transform.SetParent(poolParent);
        pool.Enqueue(obj);
    }
}
```

Example usage:
```csharp
using UnityEngine;

public class BulletPoolManager : MonoBehaviour
{
    public Bullet bulletPrefab;
    public int startAmount = 20;
    
    private ObjectPool<Bullet> bulletPool;
    
    void Awake()
    {
        bulletPool = new ObjectPool<Bullet>(bulletPrefab, startAmount, transform);
    }
    
    public Bullet GetBullet()
    {
        return bulletPool.Get();
    }
    
    public void ReturnBullet(Bullet b)
    {
        bulletPool.Return(b);
    }
}
```

Bullet Script:
```csharp
using UnityEngine;

public class Bullet : MonoBehaviour
{
    public float speed = 10f;
    public float lifeTime = 2f;
    private float timer;
    
    private BulletPoolManager poolManager;
    
    void Start()
    {
        poolManager = FindObjectOfType<BulletPoolManager>();
    }
    
    void OnEnable()
    {
        timer = lifeTime;
    }
    
    void Update()
    {
        transform.Translate(Vector3.forward * speed * Time.deltaTime);
        timer -= Time.deltaTime;
        
        if(timer <= 0)
            poolManager.ReturnBullet(this);
    }
}
```
# Object Pooling

A great way to improve performance is to use something called **Object Pooling**.
It enables us to keep a pool of the same object types so that we can call for X items and use them for Y duration before we return it to the pool.

Examples of items:
- Bullets
- Enemies
- Coins
- etc..

Instead of constantly destroying and creating items, we can create a base amount of items that we can reuse when needed to save on performance.

### Concept:

Object pooling preloads a set of GameObjects, disables them, and reuses them when needed.

This avoids memory allocation spikes and garbage collection pauses - especially important in games with frequent spawning.

--- 

### Example: Simple Pool Manager

```csharp
using System.Collections.Generic;
using UnityEngine;

public class ObjectPool : MonoBehaviour
{
    public GameObject prefab;
    public int poolSize = 10;

    private List<GameObject> pool = new List<GameObject>();

    void Start()
    {
        for (int i = 0; i < poolSize; i++)
        {
            GameObject obj = Instantiate(prefab);
            obj.SetActive(false);
            pool.Add(obj);
        }
    }

    public GameObject GetPooledObject()
    {
        foreach (var obj in pool)
        {
            if (!obj.activeInHierarchy)
            {
                obj.SetActive(true);
                return obj;
            }
        }
        // Optionally expand pool
        GameObject newObj = Instantiate(prefab);
        pool.Add(newObj);
        return newObj;
    }
}
```

### Example Usage:

```csharp
public class BulletShooter : MonoBehaviour
{
    public ObjectPool bulletPool;

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            GameObject bullet = bulletPool.GetPooledObject();
            bullet.transform.position = transform.position;
        }
    }
}
```

---

The key function of object pooling is to create objects in advance and store them in a pool, rather than have them created and destroyed on demand. When an object is needed, it’s taken from the pool and used, and when no longer needed, it’s returned to the pool rather than being destroyed. 

This does not mean that every single thing is a candidate for a pool, and you should take the pros and cons into consideration if such a thing would benefit for your project.

Unity can handle a lot of instantiation and destroying of objects, and it's okay to not use a pool for every single thing.

For example, when it comes to a bullet hell game, where you have 100s of bullet spawning in every second, it would be wise to use a pool.
But if you only create say 5 bullets every second or so and they live for 3-5 seconds, it makes no sense in having a pool.

[Link to a Unity course on pooling](https://learn.unity.com/tutorial/use-object-pooling-to-boost-performance-of-c-scripts-in-unity)
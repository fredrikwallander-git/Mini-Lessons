# Identifying Objects Through Components

Previously we've seen how you can identify objects through the use of tags or names.

While this works, it's a fragile way of identification:
- strings can be mistyped
- you are required to globally setup them in the editor
- and you cannot store extra data along with it

Instead, we are going to learn the alternative through component identification / C# type checking.

## Using TryGetComponent

`TryGetComponent<T>();` (where T == the component type you're looking for) checks if an object has a certain component.
It’s faster and cleaner than GetComponent() because it doesn’t throw exceptions or allocate memory unnecessarily.

Let's look at an example of detecting enemy collisions:

```csharp
void OnCollisionEnter2D(Collision2D collision)
{
    // The out keyword returns the object of type T if TryGetComponent returns true
    // the object can then be used safely inside this scope
    if (collision.gameObject.TryGetComponent<Enemy>(out Enemy enemy))
    {
        // We found an Enemy component!
        enemy.TakeDamage(1);
        Debug.Log("Hit enemy: " + enemy.name);
    }
}
```

No strings, no tags, just safely checking for a component of type T.

### Short info on the `out` keyword

The `out` keyword in C# is used to pass **arguments** to a method by **reference**, allowing the method to modify the variable _passed in_ and effectively return multiple values. 

Variables passed with `out` **do not need** to be **initialized** before being passed to the method, but the method **_must_** assign a value to them before it **returns**.

---

Another example is to pick up items, suppose we have:

```csharp
public class ItemPickup : MonoBehaviour
{
    public string itemName;
}
```

And in your player:

```csharp
void OnTriggerEnter2D(Collider2D other)
{
    if (other.TryGetComponent<ItemPickup>(out ItemPickup pickup))
    {
        Debug.Log("Picked up: " + pickup.itemName);
        Destroy(pickup.gameObject);
    }
}
```

With it, any object with an `ItemPickup` component will be recognized as a collectible -
you don’t need a “Collectible” **tag** or **layer**.

## Going even broader

You can combine this with an interface to make it an even broader identification.

Interface example:
```csharp
public interface IDamageable
{
    void TakeDamage(int amount);
}
```

Enemy example:
```csharp
public class Enemy : MonoBehaviour, IDamageable
{
    public int health = 3;

    public void TakeDamage(int amount)
    {
        health -= amount;
        Debug.Log($"{name} took {amount} damage. Remaining HP: {health}");
    }
}
```

Now your `Player` doesn’t need to know if it hit an `enemy`, `boss`, or `destructible crate`:

```csharp
void OnCollisionEnter2D(Collision2D collision)
{
    if (collision.gameObject.TryGetComponent<IDamageable>(out IDamageable damageable))
    {
        damageable.TakeDamage(1);
    }
}
```

Clean, scalable, and no hardcoded dependencies, which is nice.. right?
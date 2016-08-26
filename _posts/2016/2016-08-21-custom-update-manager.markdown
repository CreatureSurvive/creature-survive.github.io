---
layout: "post"
title: "Custom Update Manager"
date: "2016-08-21 19:36"
published: true
---

##### What Is A Update Manager
An Update Manager is nothing more than a list of methods that get called by iterating through that list.

---

##### Why Would You Need A Custom Update Manager

a custom update manager can offer your project increased perfomance by
lowering the time required to call `Update()` on all of your `MonoBehaviour` scripts. For example say you have 5000 GameObjects in your scene and you need to move all of them each frame.

{% highlight csharp %}
```
using UnityEngine

public class TrackPositions : MonoBehaviour
{
  void Update()
  {
    transform.Translate(Vector3.forward * Time.deltaTime);
  }
}
```
{% endhighlight %}


Running this test with unity's standard update method results in `2.08605 ms` between each `Update();` call Running the same test using this Update Manager results in `0.21295 ms` between each `Update();` call. this is due to the way unity calls its Magic Methods

---

##### How Does Unity Manage It's 'Magic Methods'?
Writing C# code in unity, is Managed Code. Managed code is code that is converted to a format that is readable by the CLR Common Language Runtime. in this case either Mono or Il2cpp would be the CLR.

native code is code that gets compiled into instructions that can be executed directly by a computer.

unity handles calling Magic Methods from native code however the methods themselves are in managed code so this causes overhead each time a method is called from native to managed code.

So in this scenario calling `Update();` on 5000 GameObjects each frame causes you to pay this overhead 5000 times, however using an Update Manager you can call Update one time in the manager and let the managed code call the 5000 update methods thus the overhead is only payed once.

---

##### Creating The Update Manager

handling the classes that will be registered with the update manager is rather simple. all we need to do is create a wrapper for MonoBehaviour. To do this all you need to do is create a new class that inherits from MonoBehaviour.

{% highlight csharp %}
```
public abstract class BaseMonoBehaviour : MonoBehaviour
{

}
```
{% endhighlight %}

in this class we want to create a virtual method that we can overload in our classes that require `Update();` to be called. we will also need to add a virtual method for `Awake();` to handle adding items to the update manager, I'll explain tis more later on.

{% highlight csharp %}
```
protected virtual void Awake() {}
public virtual void UpdateNormal() {}
```
{% endhighlight %}

now in our `MonoBehaviour` classes instead of inheriting from 'MonoBehaviour' we inherit from `BaseMonoBehaviour`.

Now we need to create the manager itself. the manager will be responsible for holding an array of all our `BaseMonoBehaviour`'s and iterating through them every frame.

so lets create a new class `UpdateManager` will need an array for our `BaseMonoBehaviour`'s

{% highlight csharp %}
```
public class UpdateManager : MonoBehaviour
{
  private BaseMonoBehaviour[] behaviours;
}
```
{% endhighlight %}

In order for this to function properly well need to have a way for our `BaseMonoBehaviour`'s to be able to communicate with our `UpdateManager` class. the easiest way of doing this is to use the constructor to declare a static instance.

```csharp
private static UpdateManager instance;

public UpdateManager()
{
    instance = this;
}
```


The first thing we'll need to do is add some functionality to to add classes to our manager, we'll handle this by adding our classes from within the `Awake()` method. The `Awake` method will be called every time a the class is created. so within our BaseMonoBehaviour class lets create a static `AddEntry() method that will call pass the `BaseMonoBehaviour` to bae added to a list

```csharp
public static AddEntry(BaseMonoBehaviour behaviour)
{
  instance.AddEntryToManager(behaviour);
}

```

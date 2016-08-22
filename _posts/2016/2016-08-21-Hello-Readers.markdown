---
title:  "Hello Readers"
date:   2016-08-21 16:14:54
description: Saying "hello" in Unity3d
published: true
---
Hello Readers

{% highlight csharp %}
using UnityEngine;

public class Hello : MonoBehaviour
{
  const string helloMSG = "Hello Readers";

  void Awake()
  {
    Debug.Log(helloMSG);
  }
}
{% endhighlight %}

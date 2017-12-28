Unity3D Good Practices
==================

_Work in progress, please contribute or fix things as you see fit_

Interested in other platforms? Our [Best Practices in Android Development][android-best-practices], [Windows App Development Best Practices][windows-app-development-best-practices] and [iOS Good Practices][ios-good-practices] documents have got you covered.

[android-best-practices]: https://github.com/futurice/android-best-practices
[windows-app-development-best-practices]: https://github.com/futurice/windows-app-development-best-practices
[ios-good-practices]: https://github.com/futurice/ios-good-practices/

## Contents

If you are looking for something specific, you can jump right into the relevant section from here.

1. [Tutorials](#tutorials)
1. [Recommended assets](#recommended-assets)
1. [Coding conventions](#coding-conventions)
1. [Optimizing](#optimizing)
1. [Testing](#testing)

## Tutorials

Here's a list of recommended tutorials:

* [Catlike Coding][catlike-coding] offers scripting basics, programmatic meshes, shaders and more.
* [Udemy][udemy-link] has paid but thorough tutorials on making entire games in Unity.

[catlike-coding]: http://catlikecoding.com/unity/tutorials/
[udemy-link]: https://www.udemy.com/unitycourse/

## Recommended assets

### DOTween - Animating from scripts

You could be spending hours upon hours writing your own [easing functions][easing-functions] in some Update-loop or UniRx Observable - or you could use a tweening library.
We recommend using [DOTween][dotween]. It's easy to learn, has a lot of functionality and it's [faster than the others][dotween-fast]!
This is extremely handy when you're trying to implement some simple UI animation for example. Do not use this for organic animation, e.g. characters. Refer to [Unity's built-in animation system][unity-animation] in that case. 

Here's a quick example on how to move something to a new position over two seconds with a quint easing, and finally logging to the console:
```csharp
transform.DOMove(new Vector3(1,2,3), 1).SetEase(Ease.InOutQuint).OnComplete(() =>
{
	Debug.Log("Hello world");
});
``` 
[easing-functions]: http://easings.net/
[dotween]: http://dotween.demigiant.com/
[dotween-fast]: http://dotween.demigiant.com/#enginesComparison
[unity-animation]: https://unity3d.com/learn/tutorials/s/animation

### TextMesh Pro - Better text for your games

Ever think 'The text in Unity is so bad...'? Worry no more, [TextMesh Pro is here to save you][textmesh-pro]
[textmesh-pro]: https://www.assetstore.unity3d.com/en/#!/content/84126

There's not much to say here, it's "just" text, done right. You can use text as a mesh in your 3D world, or use it as text for your canvas.

## Coding conventions
In this section we'll talk about raising the quality of your code. Some of these tips may affect performance, so be mindful.

### C# version
Unity builds to a runtime version of dotnet 3.5 by default.
You can switch to 4.6 by going into Player -> PC, Mac and Linux -> Other Settings -> Configuration -> Scripting Runtime Version. Here's an example of a getter property in 3.5:

```csharp
private int _foo;
public int Foo
{
	get
	{
		return _foo;
	}
}
```
And here's the same in 4.6:

```csharp
private int _foo;
public int Foo => _foo;
```

For more information, see [here][scripting-runtime-upgrade].


[scripting-runtime-upgrade]: https://docs.unity3d.com/Manual/ScriptingRuntimeUpgrade.html

### Code style
_Coming soon..._

### LINQ
Language-integrated query, or [LINQ][linq-docs] for short, is a way of handling collections in dotnet. 

Let's say you want to find a gameobject in a list that has some specific tag:
```csharp
GameObject gameObjectToFind = null;
foreach(var go in yourListOfGameObjects)
{
	if(go.tag == "fooBar")
	{
		gameObjectToFind = go;
		break;
	}
}
```

We can do this much shorter with LINQ:
```csharp
var gameObjectToFind = yourListOfGameObjects.FirstOrDefault(go => go.tag == "fooBar");
```

As with loops, you should be careful of using these sort of statements in frequently run scopes like Update or FixedUpdate. LINQ in particular allocates more memory than a trivial for-loop. Internally, LINQ still uses for-loops - but your code will be easier to read, at least once you get to know LINQ.
See more examples of LINQ in Unity [here][linq-example]


[linq-docs]: https://docs.microsoft.com/en-us/dotnet/standard/using-linq
[linq-example]: https://unity3d.college/2017/07/01/linq-unity-developers/

### Coroutines vs reactive extensions

By default, Unity offers [Coroutines][unity-coroutines] for handling future execution. In this section we'll go through the cons of that, and introduce an alternative featuring reactive programming.

Here's an example of a timed execution in Unity:


```csharp

void Start()
{
	StartCoroutine(DoOnDelay(5f));
}

IEnumerator DoOnDelay(float afterTime)
{
	yield return new WaitForSeconds(afterTime);
	// stuff that happens after time
}
```

There's a couple of problems about this. Why are we dealing with [IEnumerators][ienumerator]? What's with all this boilerplate for a simple timer?
There are better and easier ways to do this. Don't do this either though:

```csharp
bool timerStarted = false;
bool doExecuted = false;
float dt = 0;

void Start()
{
	timerStarted = true;
}

void Update()
{
	if(timerStarted)
	{
		dt += Time.deltaTime;
		if(dt >= 5f && !doExecuted)
		{
			doExecuted = true;
			// stuff that happens after time
		}
	}
}
```

This is even more boilerplate. We have to try something much more shorthand. We recommend [Reactive Extensions For Unity][unirx], UniRx for short.
You can download UniRx from the [Asset Store][unirx-asset]

Here's the same example as above, but written with UniRx:

```csharp
Observable.Timer(TimeSpan.FromSeconds(5f)).Subscribe(_ =>
{
	// stuff that happens after time
});
```

UniRx is based on [the reactive programming paradigm][rx-wiki]. In this case, we start a timer that lasts five seconds and we subscribe to that timer stream. Subscription means that we're listening to when the timer is done.
In reactive programming, whenever you do not wish to use the value handed to you from the publisher (here, the timer), you name the input parameter of your [lambda expression][lambda-expression] as underscore. 

You can also create oscillator-type timers that trigger every X TimeSpans by supplying a second parameter in timer like so:

```csharp
Observable.Timer(TimeSpan.FromSeconds(5f), TimeSpan.FromSeconds(5f)).Subscribe(_ =>
{
	// stuff that happens after every time
});
```

Please refer to the UniRx github page for more examples.

[unity-coroutines]: https://developer.apple.com/ios/human-interface-guidelines/
[ienumerator]: https://msdn.microsoft.com/en-us/library/system.collections.ienumerator(v=vs.110).aspx
[unirx]: https://github.com/neuecc/UniRx
[unirx-asset]: https://www.assetstore.unity3d.com/en/#!/content/17276
[rx-wiki]: https://en.wikipedia.org/wiki/Reactive_programming
[lambda-expression]: https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions

## Optimizing

### Object pooling
_Coming soon..._

### Profiling
_Coming soon..._

## Testing

### Unit tests
_Coming soon..._

### Platform dependent compilation
Switching platforms for sections of your scripts is done like so:
```csharp
void Start()
{
	#if UNITY_EDITOR
	Debug.Log("Hello Unity Edtor");
	#else
	Debug.Log("Hello something else");
	#endif
}
```
This is great if you want to skip parts in your application/game. See more [here][platform-compilation].


[platform-compilation]: https://docs.unity3d.com/Manual/PlatformDependentCompilation.html

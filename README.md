Unity3D Good Practices
==================

_Work in practice, please contribute or fix things as you see fit_

Interested in platforms? Our [Best Practices in Android Development][android-best-practices], [Windows App Development Best Practices][windows-app-development-best-practices] and [iOS Good Practices][ios-good-practices] documents have got you covered.

[android-best-practices]: https://github.com/futurice/android-best-practices
[windows-app-development-best-practices]: https://github.com/futurice/windows-app-development-best-practices
[ios-good-practices]: https://github.com/futurice/ios-good-practices/

## Contents

If you are looking for something specific, you can jump right into the relevant section from here.

1. [Timers and reactive programming](#timers-and-reactive-programming)

## Timers and reactive programming 

By default, Unity offers [Coroutines][unity-coroutines] for handling future execution. In this section we'll go through the cons of that, and introduce an alternative featuring reactive programming.

### Timers

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

This is even more boiler-plate. We have to try something much more shorthand. I recommend [Reactive Extensions For Unity][unirx], UniRx for short.
You can download UniRx from the [Asset Store][unirx-asset]

Here's the same example as above, but written with UniRx:

```csharp

void Start()
{
	Observable.Timer(TimeSpan.FromSeconds(5f)).Subscribe(_ =>
	{
		// do
	});
}

```

Much shorter. UniRx is based on [the reactive programming paradigm][rx-wiki]. In this case, we start a timer that lasts five seconds and we subscribe to that timer stream. Subscription means that we're listening to when the timer is done.
In reactive programming, whenever you do not wish to use the value handed to you from the publisher (here, the timer), you name the input parameter of your [lambda expression][lambda-expression] as underscore. 

[unity-coroutines]: https://developer.apple.com/ios/human-interface-guidelines/
[ienumerator]: https://msdn.microsoft.com/en-us/library/system.collections.ienumerator(v=vs.110).aspx
[unirx]: https://github.com/neuecc/UniRx
[unirx-asset]: https://www.assetstore.unity3d.com/en/#!/content/17276
[rx-wiki]: https://en.wikipedia.org/wiki/Reactive_programming
[lambda-expression]: https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions

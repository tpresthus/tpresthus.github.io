---
title: "Dependency What?"
layout: single
---

Welcome to the *Dependency What?* workshop. My name is Thomas Presthus, and I've been working as a consultant doing software development and architecture for the past ten years.

This workshop premieres at the NNUG Vestfold Kodekafé. That means that you, dear participant, is one of the first people to ever take part in it. Since this is the premiere, I ask of you to please raise any questions or problems you may encounter on your path here tonight. I further hope that we won't exceed our time slot. Furthermore I'd like to thank Verftet for hosting us - please take the opportunity to order yourself a coffee or beer to show our gratitude.

Now for the fun part! What is this workshop all about? I'd say it's mainly about the Dependency Inversion Principle, but with bits thrown in about Inversion of Control, containers (not the docker/LXC ones!) and Dependency Injection as well. (Big words, I know!)

I've found throughout the past years that while many developers kinda understands the principles behind these words, many also struggle to explain the difference between them and perhaps also to implement them in actual code.

That's why we'll try to break down the principles and approaches to implementing them, and hopefully giving a little more insight to how they help us and our codebases. 

## Prerequisities

Since this workshop is hosted by NNUG Vestfold, we'll use C# as our language of choice in code samples and excercises. The principles and patterns we present should, however, apply to most object-oriented languages. Most of them applies to functional languages as well, but those tend to have other mechanisms that might be more idiomatic (see e.g. my [blog post on Scala and the Cake Pattern](http://www.ballofcode.com/scala/ioc/cake-pattern/2014/01/26/scala-cake-pattern.html)).

You should have a functioning development environment of your choice (e.g. Visual Studio or perhaps vim and mono?). You should also have a basic understanding of object orientation.

## Dependency Inversion Principle (DIP): 101

What is the Dependency Inversion Principle (hereafter known as *DIP*)?

The principle itself was probably first described by Robert "Uncle Bob" Martin along with the [SOLID principles](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)).

It states:

> A. High-level modules should not depend on low-level modules. Both should depend on abstractions.
> 
> B. Abstractions should not depend on details. Details should depend on abstractions.

Simple enough, right? Read it again and try to dig into its actual meaning. Abstractions should not depend on details. What are these details and abstractions?

In object orientation we create abstractions and concrete implementations in order to modularize and reason about our systems. It's not always easy to come up with helpful implementations though. Consider the following example:

{% highlight csharp %}

interface IClient
{
    string FirstName { get; set; }
    string LastName { get; set; }
    IList<Address> Addresses { get; set; }

    void Activate();
    void Deactivate(string reason);

    void AddAccount(IAccount account);
    
    void Deposit(double amount);
}

{% endhighlight %}

Interfaces in C# are one kind of abstractions. They define an interface that other types can implement, and thus signal their supported operations to their consumers.

Is the above code a good abstraction though? While, yes, it's an interface and not a concrete implementation, it seems to offer nor abstraction. It's rather a means of indirection.

So what would we consider a good abstraction? We would have to abstract something away, right? Let me give you an example.

My car has automatic transmission, meaning that I don't have to manually shift gears. When I think of automatic transmission, this is what I envision:

![Shifter](../../images/shifter.jpg) 

<small>Picture by Brian Hoecht under Creative Commons [CC BY-NC-ND](https://creativecommons.org/licenses/by-nc-nd/2.0/)</small>

This is however what an automatic transmission actually looks like:

![Automatic transmission](../../images/transmission.jpg)

<small>Picture by Abdullah AlBargan under Creative Commons [CC BY-ND 2.0](https://creativecommons.org/licenses/by-nd/2.0/)</small>

See how the first image, the **shifter**, abstracts away how the automatic transmission works? Imagine the interface describing the actual transmission!

> ### Excercise
> 
> Try to define an interface for the automatic transmission shifter in your language of choice.
> 
> You could end up with something like [this](https://gist.github.com/tpresthus/07813352bf0d5d7b069d70950c665b95) when you're done.

That's probably pretty easy. Don't worry though, we'll get to more complex abstractions in a little while.

What does this abstraction give us? First of all, it's an interface that would seem natural and comprehendible by any driver. Like the driver, the abstraction would make sense to a coder trying to use your interface - allowing him not to worry about how the underlying machinery actually works.

By now you might be wondering what automatic transmissions has to do with the **DIP**. There are no dependencies in the code yet. Let's revisit the principle briefly: Code should depend on things that are at the same or higher level of abstractions.

We can draw this: ![](../../images/dependency-what/ishifter-eightspeed.png)

Notice that the IShifter abstraction has do depencies while the EightSpeedTransmission, by implementing it, has a dependency on IShifter.

Let's extend our system by adding a *Driver*: ![](../../images/dependency-what/driver-ishifter-eightspeed.png)

Notice how we now introduce another *detail*, *Driver*. The Driver has a dependency on *IShifter*. This adheres to the **DIP** since we have no abstractions depending on details. Both details are depending on an abstraction.

We could, of course, have made our Driver depend directly on the EightSpeedTransmission instead, but this would violate the **DIP**.

> ### Excercise
>
> Implement the above type system in your favorite language

That about wraps up the introductory chapter on Dependency Inversion Principle. This far we've learned that it's in fact a very simple principle that helps us create modularized and loosely coupled code by not having details depend on details.

## Inversion of Control

We've talked about the inversion of *dependencies*. Now we're going to have a look at the inversion of *control* (whatever that means, right?).

I personally think the main reason that developers struggle with DIP, DI, IoC and containers is that the naming of these concepts are so similar. I mean; Dependency *Inversion*, *Inversion of Control*, *Dependency* Injection. It all seems to overlap. I've talked to developers who thinks Dependency Inversion requires Inversion of Control Containers, or that Dependency Inversion and Dependency Injection are the same things. And aren't Inversion of Control the same as containers?

We'll come back to the container later, and focus on Inversion of Control as a pattern.

When writing code, we traditionally let the consumer of a component call it - thus being in control. This isn't necessarily a Bad Thing™, but it's all about how we do it.

Consider the following example:

{% highlight csharp %}

class Autopilot
{
    // Attempt to overtake the next car
    // by signalling a turn, shifting to a lower gear
    // and turning left (hoping that we're not in Britain)
    public async void Overtake()
    {
        var blinker = new BlinkerService();

        using(blinker.SignalLeft())
        {
            var shifter = new ShifterService(new EightSpeedTransmission());
            await shifter.GearDown();

            var steering = new SteeringService(new ServoBasedSteering());
            await steering.TurnLeft();
        }
    }
}
{% endhighlight %}

While the code is simple, it hides three major dependencies:

  - Shifter
  - Blinker
  - Steering

This makes it hard to compose this component together with the rest of our car, since as a caller we have no idea of which dependencies will be used. It also makes it harder to test.

Another problem with the code is that the Autopilot is now made responsible for both creating and maintaining the life-cycle of these depencies. We could call this classical Control (with no inversion).

Utilising Inversion of Control, we'll invert the control chain, releiving the Autopilot from the responsibility of maintaing the life-cycle for these dependencies.

There are several ways we can do this. For now, we'll use the **Service Locator** pattern. Please note that this is considered an [anti-pattern](http://blog.ploeh.dk/2010/02/03/ServiceLocatorisanAnti-Pattern/) - we will explore better alternatives a little later.

Instead of re-iterating the mechanics of the Service Locator, I'll leave it as an excercise for you to read about it [here](http://martinfowler.com/articles/injection.html#UsingAServiceLocator).

> ### Excercise
>
> Try refactoring the above code into using a Service Locator.
> You can mock out the Service Locator implementation to make
> it easier for yourself.
> 
> You could end up with something like [this](https://gist.github.com/tpresthus/4854f319e91d72167dc8616e0b69de42).

To use the Autopilot, we can now do this:

{% highlight csharp %}

void Main()
{
    ShifterService.Current = new ShifterService(new EightSpeedTransmission());
    SteeringService.Current = new SteeringService(new ServoBasedSteering());
    BlinkerService.Current = new BlinkerService();

    var autopilot = new Autopilot();
    autopilot.Overtake().ConfigureAwait(false);
}
{% endhighlight %}

Notice how the Autopilot no longer knows anything about transmission systems or how the steering works. It simply uses the dependencies that are controlled by the main loop. We have successfully inverted the control of our dependencies.

Furthermore we've adhered to the Dependency Inversion Principle of not having details depend on other details. The Autopilot detail depends on three abstractions: ShifterService, SteeringService and BlinkerService. ShifterService uses an IShifter (remember?) which we construct as an EightSpeedTransmission detail in our composition root. The same goes for the SteeringService.

Wait. Did I just say composition root? What's that? Read on to find out (or google it).

## Dependency Injection

Did you think we're finished with dependencies yet? Nope. This is where we shake it up by adding to the confusion of acronyms. We'lve talked about **DIP**, and now we're gonna look at **DI: Dependency Injection**.

As we've already seen, Dependency Inversion is about separating implementation details and abstractions. So what is Dependency Injection?



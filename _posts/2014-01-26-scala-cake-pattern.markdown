---
layout: post
title:  "The Cake Pattern"
date:   2014-01-26 17:23:00
categories: scala ioc cake-pattern
---

The past few months I've been toying around with the Scala programming language. My interest in Scala probably started with my currenct contract engaging me in developing on a Java-based stack. As I'm not a big fan of Java's code noise and verbosity, I started looking at other JVM languages. Functional Programming has also been something that I wanted to look more into, so Scala seemed like a good start.

After having done quite a lot of playing around with pattern matching and other cool aspects of Scala I got tired of writing small playground applications and decided it was about time to write something closer to a real world application. Now, the real world is saturated with complexity and complexity is a well-known application killer. Thus many smart people have defined principles for helping developers deal with complexity in simpler ways. Some of these principles have been gathered into the SOLID acronym: Single Responsibility principle, Open/Closed principle, Liskov Substitution principle, Interface Segregation principle and the **Dependency Inversion principle**.

I won't go into detail about any of these principles today, but rather talk about how we can solve Dependency Inversion in scala. Dependency Inversion can be summarized as having details depend upon abstractions (and not upon other details), as well as having abstractions not depend upon details.

Dependency Inversion is often solved through IoC - Inversion of Control, whereas Dependency Injection and Service Locating is two known patterns. The latter is considered an anti-pattern, while Dependency Injection has gained widespread usage.

Dependency Injection often looks like this in Java:

{% highlight java %}

public interface OrderRepository {
	Order getOrderBy(int number);
}

@Component
public class MysqlOrderRepository : OrderRepository {
	public Order getOrderBy(int number) {
		...
	}
}

public class UseCase {
	private final OrderRepository _orders;

	@Autowired
	public UseCase(OrderRepository orderRepository) {
		this._orders = orderRepository;
	}

	public void doSomething(int ordernumber) {
		_orders.getOrderBy(ordernumber);
	}
}
{% endhighlight %}

Here we are using Spring as an IoC container. Basically we register *MysqlOrderRepository* as a candidate for being injected as *OrderRepository*, and signal that *UseCase* needs an OrderRepository to function. When we load up an instance of *UseCase*, Spring will automagically pass an instace of MysqlOrderRepository into it.

This might look all fine and dandy, but it sure gets complicated when you start having multiple implementations that you want to use in test- and production situations. And because it's so easy to use Autowired, it's normal to end up with lots of nested dependencies. Nested dependencies are allright in some, rare scenarios. Usually, though, having lots of nested dependencies is an indication that you're doing something wrong.

In Scala, though, we can use the *Cake Pattern* to explicitly define our dependencies, and wiring them up just as we need them. I think this makes a code base far easier to read and understand, because you can immediatly see what's going on.

Let's go ahead and try implementing our example from before in Scala. First we create our OrderRepository:

{% highlight scala %}

class OrderRepository {
  def getOrderBy(number: Int): Order {
    println("Getting order by " + number)
    Order(number)
  }
}

{% endhighlight %}

Of course we could have extracted this into a trait. That would make the code look more like the Java example, but as we're dealing with a pretty simple case here, I simply didn't find it to useful to split it further.

Next up is our UseCase:

{% highlight scala %}

class UseCase {
  def doSomething(orderNumber: Int) {
    val order = orderRepository.getOrderBy(orderNumber)
    println("Doing something with " + order)
  }
}

{% endhighlight %}

As you can see, we're now referencing an *orderRepository* that has a function *getOrderBy(x: Int)*. How is this even gonna compile, you may ask? Well, it won't yet. This is where the cake pattern enters. 

First we wrap our dependency - the OrderRepository - into a trait:

{% highlight scala %}

trait OrderRepositoryComponent {
  val orderRepository: OrderRepository

  class OrderRepository {
    def getOrderBy(number: Int): Order {
      println("Getting order by " + number)
      Order(number)
    }
  }
}

{% endhighlight %}

Here we enclose the repository, and define an abstract field of OrderRepository.

Now we're gonna look at the UseCase again, which is the consumer of our dependency. In order to have our OrderRepository injected into it, we'll enclose it in a trait of it's own, and then use a self-type annotation to declare our dependency:

{% highlight scala %}

trait UseCaseComponent { 
this: OrderRepositoryComponent =>
  val useCase: UseCase

  class UseCase {
    def doSomething(orderNumber: Int) {
      val order = orderRepository.getOrderBy(orderNumber)
      println("Doing something with " + order)
    }
  }
}

{% endhighlight %}

What's interesting here is this snippet:

{% highlight scala %}
this: OrderRepositoryComponent =>
{% endhighlight %}

That's what we call a self-type annotation, and basically it means that this trait has to be mixed-in with an OrderRepositoryComponent trait.

I guess it's time to wire it all up. In order to regain some common ground, we'll wire these components together in a registry:

{% highlight scala %}

object Registry extends
  UseCaseComponent with
  OrderRepositoryComponent
{
  val orderRepository = new OrderRepository
  val useCase = new UseCase
}
{% endhighlight %}

What we've gained now, is removing the instantion of dependencies from our concrete implementations, and into a registry object. This could well be your top-level service or the like. Inside of it, we have the complete freedom and flexibility to instantiate our dependencies as we need to. 

We also make our dependencies explicit and visible from the entry-point of our code. There is no need for checking Spring XML configuration files, or having your IDE "find all implementations" of OrderRepository. We can see from the start, that orderRepository is instantiated as new OrderRepository. This is even more powerful when you have multiple implementations of OrderRepository, of course.

What's even cooler, is that we can simply compose different objects to control our dependencies in different scenarios - e.g. for testing. We can now do something like this:


{% highlight scala %}

class UseCaseSuite extends TestNGSuite 
  with UseCaseComponent 
  with OrderRepositoryComponent
{
  val orderRepository = mock(classOf[OrderRepository])

  @Test
  def doSomething() = {
    val useCase = new UseCase
    
    useCase.doSomething(5)

    // assert..
  }
}
{% endhighlight %}

Here we've mocked away our orderRepository, while creating a fresh UseCase for testing. Pretty elegant, pretty neat.

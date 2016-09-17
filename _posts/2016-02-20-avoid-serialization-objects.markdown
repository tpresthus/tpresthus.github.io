---
title: "Avoid serialization objects"
date: "2016-02-20 20:00:12 +0100"
categories: DDD boundaries serialization
---

I've long honored and practiced the idea of translating objects when their data
moves across contexts and boundaries. In this post I'm going to take a look at
how this tends to be done, and ponder about whether the wide-spread practice
of using objects to represent contracts and serializing them really is a good
solution or not.

Be it XML, JSON or practically any serialization format - when working with them
in object-oriented languages, we tend to craft objects that represent the
contract, or even protocol, of our messages. Typically, these are POCOs (C#) or
POJOs (Java). Establishing these objects makes it easy to serialize our messages,
as we can simply run them through a serializer - like e.g. Newtonsoft JSON for
.NET or Jackson for Java.

An object like this:

{% highlight c-sharp %}
class Customer
{
  public string Name { get; set; }
  public AddressDetails Address { get; set; }

  class AddressDetails
  {
    public string Street { get; set; }
    public string ZipCode { get; set; }
    public string Town { get; set; }
  }
}
{% endhighlight %}

is automatically turned into something resembling this:

{% highlight json %}
{
  "name": "John Doe",
  "address": {
    "street": "Some street",
    "zipCode": "1337",
    "town": "Our city"
  }
}
{% endhighlight %}

Seems all good, but if we poke a little at this, we'll discover why this might
actually not be what we want.

The first time you need to serialize some objects to a message, you'll probably
go ahead and feed your domain objects straight into the serializer of your choice.
Mission accomplished; you've mapped your objects to a message that can easily
be sent over the wire. After some time though, you refactor your domain objects.
Suddenly your unit tests (because you wrote them, right?) are turning red. Quite
naturally too, seeing as your message contract is effectively derived from your
domain objects - which have now been leaked through a boundary crossing.

Wise from damage, you create a new set of objects resembling your domain since
before your refactoring. You then create an Anti-Corruption Layer, or a mapper
mechanism that takes your current domain object and map it into your "new" message
object. You've now established a message contract.

I've done this myself many times, and I've even felt that it was a nice approach
because the objects belonging to the mapping domain kind of represent your
message contract. Lately, I've however started to notice that this not only
carries along extra (and unnecessary double-work), but it's also confusing for
some team members. Especially when they're not quite able to differentiate
between objects of the domain and contract objects.

Then I saw Eric Evans tweeting about something similar the other day:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/mjpt777">@mjpt777</a> <a href="https://twitter.com/Daneel3001">@Daneel3001</a> Automatic conversion of objects into DTOs or other transferable things, leads to context leaks and dumb objects.</p>&mdash; Eric Evans (@ericevans0) <a href="https://twitter.com/ericevans0/status/699347864078581761">February 15, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

I think these two tweets nail the issue. Objects encapsulate state and behavior.
Our mapping objects are simple data-bags that in turn is reflected upon by some
serializer to create a message. They have no behavior, and no state - making
them quite dumb.

So I propose another approach to implementing your messaging protocols:
Maintain the mapping code yourself in an Anti-Corruption layer. Mapping is actually
very functional: f(object) -> mappedObject. We don't need an object to represent
this when a function is enough.

This clears things up a lot, because there are no mapping objects - and so the
ambiguity surrounding contract objects versus domain objects is effectively
removed.

How might we go ahead and implement this? It depends on your platform, language
and toolbox of course. But just for the sake of it, I'll provide an example
for C# using Newtonsoft JSON.

{% highlight c-sharp %}
static JObject CreateMessageRepresentation(Customer customer)
{
  return new JObject(
    new JProperty("customer",
      new JObject(
        new JProperty("name", customer.Name),
        new JProperty("address",
          new JObject(
            new JProperty("street", customer.Address.Street),
            new JProperty("zipCode", customer.Address.Zip),
            new JProperty("town", customer.Address.City)
          )
        )
      )
    )
  )
}

var message = CreateMessageRepresentation(someCustomer);
Console.WriteLine(message);
{% endhighlight %}

This gives us something like:

{% highlight json %}
{
  "customer": {
    "name": "John Doe",
    "address": {
      "street": "Some street",
      "zipCode": "1337",
      "town": "Our city"
    }
  }
}
{% endhighlight %}

Note how explicit the mapping is. This gives you total control of how the message
is produced, without relying on the reflection-abilities of a serializer.

---
layout: post
title: "Avoid serialization objects"
date: "2016-02-19 00:00:12 +0100"
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

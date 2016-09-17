---
title:  "Tell, don't ask!"
date:   2016-03-23 22:00:00
categories: principles craftmanship coding
---

**Tell, don't ask** is a principle commonly used in object-oriented programming. It guides our designs towards less coupling and 
less ambiguity in our interfaces.

Object-oriented programming was meant to be objects sending each other messages. Today we know this as objects invoking methods on each
other. I feel that many programmers have yet to grasp the idea of message-sending, which leads them to writing code like this:

{% highlight python %}
condition = other_object.get_state()
if condition.foo == "bar":
  other_object.do_this()
else:
  other_object.do_that()
{% endhighlight %}

This might, at first glance, seem fine. But what happens when the list of conditions grow? Or what happens if _other object_ is used
from another place?

We're basically breaking a couple of principles here. Most importantly, we're breaking encapsulation. OtherObject is no longer in charge
of its invariants, since we're peeking into it and doing operations based on how we interpret its state. What happens if we make changes
to OtherObject? We'll probably break something, as we have other objects depending on its internal state.

Another issue caused by this kind of code is that of readability. A new consumer of OtherObject would have to check out the source for
OtherObject before using it, because its exposed interface doesn't tell the consumer how to use it.

If we instead apply the **Tell, don't ask principle** our code would take on a shape similar to this:

{% highlight python %}
class OtherObject:
  state = {..}
  
  def do_your_job(self):
    if state.foo == "bar":
      self.do_this()
    else:
      self.do_that()
  
  {..}
  
class CallingObject:
  def __main__():
    other_object = OtherObject()
    other_object.do_your_job()
    
{% endhighlight %}

Now our calling object doesn't have to peek into OtherObject to determine what to do. We've encapsulated the behavior with its associated
data in the object owning it. Furthermore we've effectively applied the **Tell, don't ask** principle by letting our calling object
trust the other object to do its job.

The code reads better, is easier to understand and ensures that we can refactor the internals of OtherObject however we like, as long
as we maintain that simple interface of "do_your_job()".

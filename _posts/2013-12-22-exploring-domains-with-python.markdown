---
layout: post
title:  "Exploring domains with python"
date:   2013-12-22 17:19:00
categories: python domain-driven-design
---

I'm currently working on a Proof of Concept project. But the concept isn't really finished. This leads to a rather fun path of development: Discovering the concept and exploring its domain(s) at the same time.

The concept I'm prototyping has rather complex business logic and requirements, and we've decided to use domain-driven design in developing it. Domain-driven design lets us define clear business goals and mechanisms, while implementing fitting software based on the actual business needs and domains. The product is further broken down into (at the moment) 2 different bounded contexts. I'll write more about bounded contexts in another post.

I've chosen to start coding in the bounded context that contains our primary domain. When prototyping like this, I don't want to get bugged down by strict infrastructure, limiting frameworks or anything else. I just want to start sketching out my models in actual code. I've found python to be a great tool for this kind of work. A dynamic, verbose language with a full-packed toolkit of standard libraries empowers me to get up and running quickly.

###Repositories

A well-known tactical pattern of domain-driven design is the Repository. A repository encapsulates the storage and retrieval of our domain objects. Be it towards a database, event store, web service or frankly anything.

When prototyping, I like to keep the number of dependencies in my project down. This also adheres to one of my rules of good architecture: Good architecture lets you defer concrete decisions. E.g. allowing you to code up your domain without thinking about databases and those kind of things. Repositories serves as a nice seam in our code to isolate the domain from this kind of concerns. Another goal of domain-driven design is also to keep development focused on the model instead of technology.

One of my best friends in the python standard library is pickle. Pickle lets us serialize and deserialize nearly any python object or structure.

By using pickle, an example repository might look like this:

{% highlight python %}
import pickle

class AccountRepository:
  def get(self, account_name):
    return pickle.load(open("account-%s" % account_name, "rb"))

  def store(self, account):
    pickle.dump(open("account-%s" % account_name, "wb"))

{% endhighlight %}

Of course, this code could easily be refactored into a pickle-backed generic repository, serving as a quick-to-use building block in our prototype project so we won't have to worry about storage again anytime soon.

{% highlight python %}
class PickleStorage:
  def __init__(self, path):
    self.path = path

  def open_file(self, filename, mode):
    return open(os.path.join(self.path, filename), mode)

  def get(self, filename):
    with self.open_file(filename, "rb") as f:
        obj = pickle.load(f)
        return obj

  def store(self, obj, filename):
    with self.open_file(filname, "wb") as f:
        pickle.dump(obj, f)

class AccountRepository:
  def __init__(self, storage):
    self.storage = storage

  def get(self, account_name):
    return self.storage.get("account-%s" % account_name)

  def store(self, account):
    self.storage.store(account, "account-%s" % account.name)
{% endhighlight %}

Now we've simply encapsulated the lifecycle management of our Account aggregate in an AccountRepository that can be replaced when the need arises. Using duck-typing, we can also switch out the storage mechanism very easily when writing tests.

###Summary

This has been a short introduction to Repositories and how we can use them when prototyping in python. I expect to write about other concepts from domain-driven design, and other ways of utilizing python for prototyping, in future posts.

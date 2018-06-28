---
title:  "Breaking the monolith - without microservices"
date:   2018-06-01 12:00:00
categories: micro-service talk cleaning-up-monliths ddd
header:
  overlay_image: ddd-exchange.jpg
  caption: "Photo by: **SkillsMatter** [All rights reserved](https://www.flickr.com/photos/skillsmatter/27918127278/in/album-72157668270763378/)"
---

Back in April I got the chance to present my talk "Cleaning up an unwieldy monolith without using microservices" at DDD eXchange 2018 in London. I'm honored by the opportunity I got, and had a blast talking about this subject - which is very dear to my heart.

You can see the recording, and slides, [of the presentation here](https://skillsmatter.com/skillscasts/11499-cleaning-up-an-unwieldy-monolith-without-using-microservices). Signing up is free, and you get to see all the other awesome talks as well!

As practitioners of domain-driven design we want to model and align our software with the business, but many of us experience how our legacy codebases holds us back. They provide value, but are often constraining, making it harder to apply domain-driven design. Have you not fantasized about the greenfield rewrite with proper models and abstractions, only to realize how hopeless it would be? Enter microservices: The promised land and seemingly silver bullet for tidying up an unwieldy monolith. They certainly appear tempting, but aren’t suitable for all of us, and cleaning up a messy monolith really isn’t their job. Eric Evans proposed the notion of bubble contexts back in 2012 as a means to create space for modelling in a legacy system. Perhaps we could take a step back and build a better understanding of our monoliths’ boundaries using this heritage.

This case study will look into how we started our journey of cleaning up a mission-critical monolith at a major Scandinavian payment solutions provider using bubble contexts and other aspects from domain-driven design. Using these we found a low-risk method of attacking our monolith and how to structure it based on our newfound insights of the domain.

You will learn how we took control over our monolith using DDD, why we chose not to use microservices - as well as both the failures and successes we had. You will also learn more technical details of some of the techniques we employed, and how we went about to adjust our surrounding organization to start thinking about business and development as one.

## Who am I?

Thomas is a consultant from Norway who specializes in software architecture and development. He's been a practitioner of Domain-Driven Design for the past 7 years or so and finds great joy in pondering in business problems. Clients and colleagues know him as an energetic and passionate craftsman who loves to learn, experiment, fail and succeed while sharing his own experiences and knowledge. Having worked with too many languages and technologies to mention, Thomas has found the intersection between business and IT to be a far more rewarding approach to problem solving and easing up the everyday work of software development. He's known for holding workshops and talks for both his clients and at local user groups.

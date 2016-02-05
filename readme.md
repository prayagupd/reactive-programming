# Principles of Reactive Programming
These are my notes and study guide how I approach studying `Principles of Reactive Programming` from [Coursera](https://class.coursera.org/reactive-002).

[![Build Status](https://travis-ci.org/dnvriend/reactive-programming.svg)](https://travis-ci.org/dnvriend/reactive-programming)

# What is Reactive Programming (RP)?
> The basic principle of reactive programming is: `Reacting to sequence of events that happen in time`, 
> and, using these patterns to, build software systems that are more robust, more resilient, more flexible and 
> better positioned to meet modern demands. -- <cite>[Reactive Manifesto](http://www.reactivemanifesto.org/)</cite>

> In computing, reactive programming is a `programming paradigm oriented around data flows and the propagation of 
> change`. This means that it should be possible to express static or dynamic data flows with ease in the 
> programming languages used, and that the underlying execution model will automatically propagate changes through 
> the data flow. -- <cite>[Wikipedia](http://en.wikipedia.org/wiki/Reactive_programming)</cite>

- [Youtube: Erik Meijer - What does it mean to be Reactive?](https://www.youtube.com/watch?v=sTSQlYX5DU0)
- [Youtube: Dr. Roland Kuhn - Go Reactive at the Trivento Summercamp](https://www.youtube.com/watch?v=auYuWBudVt8)
- [Slideware: Lutz Hühnken - A Pragmatic View of Reactive](http://www.slideshare.net/lutzh/a-pragmatic-view-of-reactive)
- [Slideware: Jonas Boner - Life Beyond the illusion of present](http://www.slideshare.net/jboner/life-beyond-the-illusion-of-present)
- [Slideware: Takipi - Advanced Production Debugging](http://www.slideshare.net/Takipi/advanced-production-debugging)
- [Mathias - Reactive Streams: The Now](http://spray.io/scaladays/2015/#/)


# Akka

## Documentation
- [The Akka Roadmap](https://docs.google.com/document/d/18W9-fKs55wiFNjXL9q50PYOnR7-nnsImzJqHOPPbM4E/pub)
- [Akka News](http://akka.io/news/)
- [Akka 2.3 -> 2.4 Migration Guide](http://doc.akka.io/docs/akka/snapshot/project/migration-guide-2.3.x-2.4.x.html)

# Hystrix
> Hystrix is a latency and fault tolerance library designed to isolate points of access to remote systems, services and 3rd party libraries, stop cascading failure and enable resilience in complex distributed systems where failure is inevitable.
-- <cite>[Hystrix - GitHub](https://github.com/Netflix/Hystrix)</cite>

> Applications in complex distributed architectures have dozens of dependencies, each of which will inevitably fail at some point. If the host application is not isolated from these external failures, it risks being taken down with them.
-- <cite>[Hystrix Wiki](https://github.com/Netflix/Hystrix/wiki)</cite>

> Hystrix is not about Futures and Promises, it is about bulk-heading and isolating dependencies by limiting concurrent execution, circuit breakers, real time monitoring and metrics. Futures are just a mechanism by which async execution is exposed. Futures by themselves do not provide the same degree of fault-tolerance functionality (though parts of it can be achieved with careful use of timeouts and thread-pool sizing). You could think of Hystrix as a hardened extension of a Future.
-- <cite>[Google Groups](https://groups.google.com/forum/#!topic/hystrixoss/jAL3tV9lc30)</cite>

Note; it basically focusses on the [Resilient](http://www.reactivemanifesto.org/) part of Reactive applications.

## Documentation
- [Netflix - Introducing Hystrix for Resilience Engineering](http://techblog.netflix.com/2012/11/hystrix.html)
- [Enonic - Resilience with Hystrix](http://labs.enonic.com/articles/resilience-with-hystrix)
- [Ben Christensen - Application Resilience in a Service-Oriented Architecture using Hystrix](http://benjchristensen.com/2013/06/10/application-resilience-in-a-service-oriented-architecture-using-hystrix/)
- [Ben Christensen - Application Resilience Engineering and Operations at Netflix with Hystrix - JavaOne 2013](https://speakerdeck.com/benjchristensen/application-resilience-engineering-and-operations-at-netflix-with-hystrix-javaone-2013)

# GitHub Markdown
- [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

# Parleys
- [Scala Days 2015 - San Fransisco](https://www.parleys.com/channel/scala-days-san-francisco-2015)
- [Scala Days 2014 - Berlin](https://www.parleys.com/channel/scala-days-2014)
- [Scala Days 2013 - New York](https://www.parleys.com/channel/scaladays-2013)

# Number of students enrolled
On 2015-04-28 there were `23,200` students enrolled, which is roughly `12,000` less than last time (2014 edition). 

# Scala
- The [Scala Source Code on GitHub](https://github.com/scala/scala)
- The [Scala API Docs for 2.11.6](http://www.scala-lang.org/files/archive/api/2.11.6/#package)

# Scalaz
- [Scalaz - Scalaz](https://github.com/scalaz/scalaz) a Scala library for functional programming.
- [Learning Scalaz](http://eed3si9n.com/learning-scalaz/)

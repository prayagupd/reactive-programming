# Week 6: Supervision and Akka Persistence
> `Supervision` describes a dependency relationship between actors: the supervisor delegates tasks to subordinates and therefore must respond to their failures.
-- <quote>[Akka.io - Supervision and Monitoring](http://doc.akka.io/docs/akka/2.3.11/general/supervision.html)</quote>

> `Akka persistence` enables `stateful actors` to persist their internal state so that it can be recovered when an actor is started, restarted after a JVM crash or by a `supervisor`, or migrated in a cluster. The key concept behind Akka persistence is that only changes to an actor's internal state are persisted but never its current state directly (except for optional snapshots). These changes are only ever appended to storage, nothing is ever mutated, which allows for very high transaction rates and efficient replication. Stateful actors are recovered by replaying stored changes to these actors from which they can rebuild internal state. This can be either the full history of changes or starting from a snapshot which can dramatically reduce recovery times. Akka persistence also provides point-to-point communication with `at-least-once message delivery` semantics.
-- <quote>[Akka.io - Persistence](http://doc.akka.io/docs/akka/2.3.11/scala/persistence.html)</quote>

## Plugins
Akka persistence uses persistence plugins to store the `event` journal entries or `snapshot` 'current-state' entries into a persistence store or log. There are a lot of plugins available on the [Akka Community Projects](http://akka.io/community/) site, the most notable ones are:

- [GitHub - akka-persistence-jdbc by myself](https://github.com/dnvriend/akka-persistence-jdbc)
- [GitHub - akka-persistence-inmemory by myself](https://github.com/dnvriend/akka-persistence-inmemory) - Handy for testing purposes
- [GitHub - akka-persistence-cassandra by Martin Krasser](https://github.com/krasserm/akka-persistence-cassandra/)
- [GitHub - akka-persistence-kafka by Martin Krasser](https://github.com/krasserm/akka-persistence-kafka/)

## Video
- [Parleys - Heiko Seeberger - Akka Overview and State Machines](https://www.parleys.com/tutorial/heiko-seeberger-akka-overview-state-machines)
- [Parleys - Nicolas Jozwiak - Supervise your Akka actors](https://www.parleys.com/tutorial/supervise-your-akka-actors)
- [Parleys - Konrad Malawski - Resilient applications with Akka Persistence](https://www.parleys.com/tutorial/resilient-applications-akka-persistence)
- [Youtube - Intro to Akka persistence with Patrik Nordwall](https://www.youtube.com/watch?v=r5lecCBazvE)

## Blogs
- [Xebia - Raymond Roestenburg - Supervisor Strategy Using An Exponential Back Off Algorithm](http://blog.xebia.com/2012/12/12/exponential-backoff-with-akka-actors/)

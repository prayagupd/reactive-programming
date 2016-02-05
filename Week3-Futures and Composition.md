# Week 3 - Futures and Composition
> Futures provide a nice way to reason about performing many operations in parallel– in an efficient and 
> non-blocking way. The idea is simple, a Future is a sort of a placeholder object that you can create for a result 
> that does not yet exist. Generally, the result of the Future is computed concurrently and can be later collected. 
> Composing concurrent tasks in this way tends to result in faster, asynchronous, non-blocking parallel code. 
-- <cite>[ScalaDocs](http://docs.scala-lang.org/overviews/core/futures.html)</cite>

## Async-Await
- [Scala - Async await](https://github.com/scala/async)
- [Scala - SIP-22 - Async](http://docs.scala-lang.org/sips/pending/async.html)
- [The Future is not good enough: coding with async/await](http://engineering.roundupapp.co/the-future-is-not-good-enough-coding-with-async-await/)

## Futures
- [Scala - Futures](http://docs.scala-lang.org/overviews/core/futures.html)
- [Akka  - Futures](http://doc.akka.io/docs/akka/2.3.10/scala/futures.html)
- [The Neophyte's Guide to Scala Part 8 - Welcome to the Future](http://danielwestheide.com/blog/2013/01/09/the-neophytes-guide-to-scala-part-8-welcome-to-the-future.html)
- [The Neophyte's Guide to Scala Part 9 - Promises and Futures in Practice](http://danielwestheide.com/blog/2013/01/16/the-neophytes-guide-to-scala-part-9-promises-and-futures-in-practice.html)

## Notes from the previous year
- [Ian Irvine](https://github.com/iirvine) - [Notes (2014) - Monads and Effects](https://github.com/iirvine/principles-of-reactive-programming/blob/master/notes/week-3/001-monads-and-effects.md)
- [Ian Irvine](https://github.com/iirvine) - [Notes (2014) - Latency as an Effects](https://github.com/iirvine/principles-of-reactive-programming/blob/master/notes/week-3/002-latency-as-an-effect.md)
- [Ian Irvine](https://github.com/iirvine) - [Notes (2014) - Combinators on Futures](https://github.com/iirvine/principles-of-reactive-programming/blob/master/notes/week-3/003-combinators-on-futures.md)
- [Ian Irvine](https://github.com/iirvine) - [Notes (2014) - Composing Futures](https://github.com/iirvine/principles-of-reactive-programming/blob/master/notes/week-3/004-composing-futures.md)
- [Ian Irvine](https://github.com/iirvine) - [Notes (2014) - Promises](https://github.com/iirvine/principles-of-reactive-programming/blob/master/notes/week-3/005-promises.md)

## Hint 1: Future combinators
Most combinators are explained in the videos by Eric Meijer. Please view these videos again and implement them in the
nodescala package object. Some combinators are already available in the Future object itself, so if you are lazy, you
can reuse those.

## Hint 2: A future that does not complete
Does a `Promise[T]().future` complete?

## Hint 3: Launching the web server
The timeout looks a whole lot like the `userInterrupted` future structure

## Hint 4: TerminatedRequested Future
Reuse the future `userInterrupted` and `timeout`. When `any` of those fail, the `terminatedRequested` future should fail.

## Hint 5: Unsubscribe from the server
Note that to cancel a Future, you should use the `val subscription: Subscription = Future.run() { (ct: CancellationToken) => }` future construct.  
The `Future.run() { ct => }` construct returns a `Subscription` that can be used to `unsubscribe` from. When you call
`subscription.unsubscribe`, the `CancellationToken`, that is available in the curried function (the `context` if you will), 
that contains the members `isCancelled: Boolean` and `nonCancelled = !isCancelled` properties, can be queried to figure out
whether or not the future has been canceled. 

So the question remains, how does one `unsubscribe`, the thereby cancel all requests that the server handles, when a 
`subscription` is in scope? 

## Hint 6: Creating the response
The `respond` method, that will be called by the `start` method (you should implement both), will stream the result back
using the `exchange`'s `write` method. 

In a loop, you should check whether or not the `token` has been canceled.

In a loop, you should check whether or not the `response` has more Strings; it is an Iterator.

After you're done writing to the `exchange`, please close the stream using `exchange.close()`

## Hint 7: The solution
The solution is:

```scala
private def respond(exchange: Exchange, token: CancellationToken, response: Response): Unit = {
  while(response.hasNext && token.nonCancelled) {
    exchange.write(response.next())
  }
  exchange.close()
}
```

## Hint 8: The start method
Eric Meijer likes the `async-await` construct, because you can use imperative constructs together with async constructs
but still being non-blocking. The `while(ct.nonCanceled)` 'problem' makes it imperative.

## Hint 9: The start method
You should first create a `listener`, then `start` the listener, which returns a `Subscription`. Then you should 
create a cancellable context using the `Future.run() {}` construct. You should then `loop` while the context is nonCanceled,
then you should wait for a nextRequest from the listener, then you should respond
 
## Hint 10: The start method
Return a combined `Subscription` with the `Subscription(subscription1, subscription2)` construct.

## Hint 11: The start method
You can respond by applying the `handler` with the `request`, which gives you a `Response`, and calling the `respond` method.

## Hint 12: The solution
The solution is:

```scala
def start(relativePath: String)(handler: Request => Response): Subscription = {
  val listener = createListener(relativePath)
  val listenerSubscription: Subscription = listener.start()
  val requestSubscription: Subscription = Future.run() { (ct: CancellationToken) =>
    async {
      while (ct.nonCancelled) {
        val (req, exch) = await (listener.nextRequest())
        respond(exch, ct, handler(req))
      }
    }
  }
  Subscription(listenerSubscription, requestSubscription)
}
```

## Hint 13: The solution
You could probably rewrite the solution to:

```scala
def start(relativePath: String)(handler: Request => Response): Subscription = {
  val listener = createListener(relativePath)
  val listenerSubscription: Subscription = listener.start()
  val requestSubscription: Subscription = Future.run() { (ct: CancellationToken) =>
    Future {
      while (ct.nonCancelled) {
        Await.result(listener.nextRequest().map { req =>
          respond(req._2, ct, handler(req._1))
        }, Duration.Inf)
      }
    }
  }
  Subscription(listenerSubscription, requestSubscription)
}
```

or

```scala
def start(relativePath: String)(handler: Request => Response): Subscription = {
  val listener = createListener(relativePath)
  val listenerSubscription: Subscription = listener.start()
  val requestSubscription: Subscription = Future.run() { (ct: CancellationToken) =>
    Future {
      while (ct.nonCancelled) {
        Await.result(listener.nextRequest().map {
          case (req: Request, exch: Exchange ) => respond(exch, ct, handler(req))
        }, Duration.Inf)
      }
    }
  }
  Subscription(listenerSubscription, requestSubscription)
}
```

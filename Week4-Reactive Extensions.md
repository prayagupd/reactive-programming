# Week 4: Rx: Reactive Extensions
> Users expect real time data. They want their tweets now. Their order confirmed now. They need prices accurate as of now. Their online games need to be responsive. As a developer, you demand fire-and-forget messaging. You don't want to be blocked waiting for a result. You want to have the result `pushed` to you when it is ready. Even better, when working with result sets, you want to receive individual results as they are ready. You do not want to wait for the entire set to be processed before you see the first row. The world has moved to `push`; users are waiting for us to catch up. Developers have tools to `push` data, this is easy. Developers need tools to `react to push data`. 
-- <cite>[Introduction to Rx](http://www.introtorx.com/Content/v1.0.10621.0/01_WhyRx.html#WhyRx)</cite>

> Rx offers a natural paradigm for dealing with sequences of events. A sequence can contain zero or more events. Rx proves to be most valuable when `composing sequences of events`. 
-- <cite>[Introduction to Rx](http://www.introtorx.com/Content/v1.0.10621.0/01_WhyRx.html#WhyRx)</cite>

> You can think of Rx as providing an API similar to Java 8 / Groovy / Scala collections (methods like filter, forEach, map, reduce, zip etc) - but which operates on an asynchronous stream of events rather than a collection. So you could think of Rx as like working with asynchronous `push-based` collections (rather than the traditional synchronous pull based collections). 
-- <cite>[Camel Rx](http://camel.apache.org/rx.html)</cite>

Please note that Rx focusses on `push-based` events. There is no way for the network to go from a `push-based` model to a `pull-based` model like with [Reactive Streams](http://www.reactive-streams.org/), because the network has no notion of an upstream (the demand stream), in which the `subscriber` communicates its demand for data to the `publisher`. With Rx there is only a downwards stream, in which the `publisher` pushes the data-items to the `subscribers`. The [RxJavaReactiveStreams](https://github.com/ReactiveX/RxJavaReactiveStreams) project makes Rx compatible with [Reactive Streams](http://www.reactive-streams.org/).

## Video
- [A Playful Introduction to Rx - Erik Meijer](https://www.youtube.com/watch?v=WKore-AkisY)
- [RxJava: Reactive Extensions in Scala](https://www.youtube.com/watch?v=tOMK_FYJREw)
- [Ben Christensen - "Functional Reactive Programming with RxJava](https://www.youtube.com/watch?v=_t06LRX0DV0)
- [DevCamp 2010 Keynote - Rx: Curing your asynchronous programming blues](http://channel9.msdn.com/Blogs/codefest/DC2010T0100-Keynote-Rx-curing-your-asynchronous-programming-blues)
- [Channel 9 - Rx Workshop: Introduction](http://channel9.msdn.com/Series/Rx-Workshop/Rx-Workshop-Introduction)
- [An Event-driven and Reactive Future - Jonathan Worthington](https://www.youtube.com/watch?v=_VdIQTtRkb8)

## Books
- [Free online - Introduction to Rx](http://www.introtorx.com/Content/v1.0.10621.0/01_WhyRx.html#WhyRx)

## Docs
- [Rx - Operators Reference](http://reactivex.io/documentation/operators.html)
- [RxJava <-> RxScala API](http://reactivex.io/rxscala/comparison.html)
- [ReactiveX - Portal](http://reactivex.io/)
- [RxScala](https://github.com/ReactiveX/RxScala)
- [RxJava](https://github.com/ReactiveX/RxJava)
- [RxJava Wiki](https://github.com/ReactiveX/RxJava/wiki)
- [Microsoft Open Technologies - Rx](https://rx.codeplex.com/)
- [The Rx Observable](http://reactivex.io/documentation/observable.html)
- [Reactive Programming in the Netflix API with RxJava](http://techblog.netflix.com/2013/02/rxjava-netflix-api.html)
- [Lee Campbell - Reactive Extensions for .NET an Introduction](http://leecampbell.blogspot.co.uk/2010/08/reactive-extensions-for-net.html)
- [MSDN - The Reactive Extensions (Rx)...](https://msdn.microsoft.com/en-us/data/gg577609)

## Docs
- [Scala Swing Docs 2.11.1](http://www.scala-lang.org/api/2.11.1/scala-swing/#scala.swing.package)

## Hint 1
The textValues and clicks observables:

```scala
 def textValues: Observable[String] =
      Observable[String]({ subscriber =>
        val eventHandler: PartialFunction[Event, Unit] = {
          case ValueChanged(source) =>
            subscriber.onNext(source.text)
        }
        field.subscribe(eventHandler)
        Subscription {
          field.unsubscribe(eventHandler)
        }
      })  
```

```scala
 def clicks: Observable[Button] =
      Observable[Button] ({ subscriber =>
        val eventHandler: PartialFunction[Event, Unit] = {
          case ButtonClicked(source) =>
            subscriber.onNext(source)
        }
        button.subscribe(eventHandler)
        Subscription {
          button.unsubscribe(eventHandler)
        }
      })
```

## Hint 2
Wikipedia API:

```scala
def sanitized: Observable[String] =
   obs.map(_.replaceAll(" ", "_"))
      
def recovered: Observable[Try[T]] =
   obs.map(Try(_)).onErrorReturn(Failure(_))
   
def timedOut(totalSec: Long): Observable[T] =
  obs.take(totalSec.seconds)

def concatRecovered[S](requestMethod: T => Observable[S]): Observable[Try[S]] =
  obs.flatMap(requestMethod(_).recovered)
```

## Hint 3
WikipediaSuggest.scala

```scala
    val searchTerms: Observable[String] =
      searchTermField.textValues

    val suggestions: Observable[Try[List[String]]] =
      searchTerms.flatMap(wikiSuggestResponseStream).recovered

    val suggestionSubscription: Subscription =
      suggestions.observeOn(eventScheduler).subscribe { (x: Try[List[String]]) =>
        x.map(suggestionList.listData = _)
          .recover { case t: Throwable =>
          status.text = t.getLocalizedMessage
         }
      }

    val selections: Observable[String] =
      button.clicks.flatMap { _ =>
        suggestionList.selection.items.self match {
               case seq: Seq[String] if seq.nonEmpty =>
                Observable.just(seq.head)
               case _ =>
                Observable.empty
          }
      }

    val pages: Observable[Try[String]] =
      selections.flatMap(wikiPageResponseStream).recovered

    val pageSubscription: Subscription =
      pages.observeOn(eventScheduler) subscribe { (x: Try[String]) =>
        x.map (editorpane.text = _)
         .recover { case t: Throwable =>
            status.text = t.getLocalizedMessage
          }
      }
  }
```

## Video
- [Promise of the Futures](https://www.youtube.com/results?search_query=scala+futures)
- [Composable Futures with Akka 2.0 - Mike Slinn](https://www.youtube.com/watch?v=VCattsfHR4o)

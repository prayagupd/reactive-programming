# Week 2 - Functional Reactive Programming
> Functional reactive programming (FRP) is a programming paradigm for reactive programming (asynchronous dataflow 
> programming) using the building blocks of functional programming (e.g. map, reduce, filter). FRP has been used for 
> programming graphical user interfaces (GUIs), robotics, and music, aiming to simplify these problems by explicitly > modeling time. 
-- <cite>[Wikipedia](http://en.wikipedia.org/wiki/Functional_reactive_programming)</cite>

I would advice reading / viewing the resources below to get a good idea on what 
[Functional Reactive Programming](http://en.wikipedia.org/wiki/Functional_reactive_programming) is. The model we use 
this week is push based, in which systems take events and push them through a 'signal' network to achieve a result. The basic
idea of FRP that we focus on this week is that events are combined into 'signals' that always have a current value, but change discretely.
The changes are event-driven. But instead of having an event handler that returns Unit, (like the onClick handler and such), it returns
a value. 

FRP in a nutshell (for now at least):

When we do an assignment in Scala, the following happens:

```scala
scala> var a = 1
a: Int = 1

scala> var b = 2
b: Int = 2

scala> var c = a + b
c: Int = 3

scala> a = 2
a: Int = 2

scala> c
res1: Int = 3

scala> var c = a + b
c: Int = 4
```

As we can see, the value of `c` did not change, when we changed the value of `a` from `1` to `2`. This is normal behavior
because we have expressed the relationship at one point in the execution of the program. 

But what if, `c` would change when we changed the value of a dependent value like `a`. This would mean that there is a 
`dependency` created between `c`, `a` and `b` that expresses how these values will relate over time. So the basic idea is 
that `c` will change when we change either `a` and/or `b`.

## Hint 1: TweetText
The following should work:

```scala
 def tweetRemainingCharsCount(tweetText: Signal[String]): Signal[Int] =
    Signal(MaxTweetLength - tweetLength(tweetText()))

  def colorForRemainingCharsCount(remainingCharsCount: Signal[Int]): Signal[String] =
    Signal {
      remainingCharsCount() match {
        case count if (0 to 14).contains(count) => "orange"
        case count if count < 0 => "red"
        case _ => "green"
      }
    }
```

## Hint 2: Polynomal
Please first try it yourself, then if you wish, verify.

```scala
  def computeDelta(a: Signal[Double], b: Signal[Double], c: Signal[Double]): Signal[Double] =
  Signal {
    Math.pow(b(), 2) - (4 * a() * c())
  }

  def computeSolutions(a: Signal[Double], b: Signal[Double], c: Signal[Double], delta: Signal[Double]): Signal[Set[Double]] =
    Signal {
      delta() match {
        case discriminant if discriminant < 0 => Set()
        case discriminant if discriminant == 0 => Set(calcLeft(a(), b(), c()))
        case discriminant => Set(calcLeft(a(), b(), c()), calcRight(a(), b(), c()))
      }
    }

  def calcLeft(a: Double, b: Double, c: Double): Double =
    (-1 * b + Math.sqrt(Math.pow(b, 2) - (4 * a * c))) / (2 * a)

  def calcRight(a: Double, b: Double, c: Double): Double =
    (-1 * b - Math.sqrt(Math.pow(b, 2) - (4 * a * c))) / (2 * a)
```

## Hint 3: Calculator
Please first try it yourself, then if you wish, verify.

```scala
  def computeValues(namedExpressions: Map[String, Signal[Expr]]): Map[String, Signal[Double]] = {
    namedExpressions.mapValues { expr =>
      Signal(eval(expr(), namedExpressions))
    }
  }

  def eval(expr: Expr, references: Map[String, Signal[Expr]]): Double = {
    expr match {
      case Literal(v) => v
      case Ref(name) => eval(getReferenceExpr(name, references), references - name)
      case Plus(aExpr, bExpr)   => eval(aExpr, references) + eval(bExpr, references)
      case Minus(aExpr, bExpr)  => eval(aExpr, references) - eval(bExpr, references)
      case Times(aExpr, bExpr)  => eval(aExpr, references) * eval(bExpr, references)
      case Divide(aExpr, bExpr) => eval(aExpr, references) / eval(bExpr, references)
      case _ => Double.MaxValue
    }
  }

  /** Get the Expr for a referenced variables.
   *  If the variable is not known, returns a literal NaN.
   */
  private def getReferenceExpr(name: String, references: Map[String, Signal[Expr]]): Expr = {
    references.get(name).fold[Expr](Literal(Double.NaN)) {
      exprSignal => exprSignal()
    }
  }
```

## Documentation
- [What is the difference between view, stream and iterator?](http://docs.scala-lang.org/tutorials/FAQ/stream-view-iterator.html)
- [Wikipedia - Functional Reactive Programming](http://en.wikipedia.org/wiki/Functional_reactive_programming)
- [Functional Reactive Animation - Elliott / Hudak (PDF)](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.52.2850&rep=rep1&type=pdf)
- [Deprecating the Observer Pattern - Odersky / Maier (PDF)](http://infoscience.epfl.ch/record/176887/files/DeprecatingObservers2012.pdf)
- [Stackoverflow - What happened to scala.react?](http://stackoverflow.com/questions/21546456/what-happened-to-scala-react)

## FRP Libraries
- [scala.frp](https://github.com/dylemma/scala.frp) - [Dylan Halperin](https://github.com/dylemma) 
- [scala.react](https://github.com/dylemma/scala.react) - [Dylan Halperin](https://github.com/dylemma)
- [React4J](https://bitbucket.org/yann_caron/react4j/wiki/Home) - [Yann Caron](https://bitbucket.org/yann_caron)

## Books
- [Reactive Design Patterns - Kuhn](http://manning.com/kuhn/) - [Chapter 1 (PDF)](http://manning.com/kuhn/RDP_meap_CH01.pdf)
- [Functional Reactive Programming - Blackheath](http://www.manning.com/blackheath/) - [Chapter 1 (PDF)](http://www.manning.com/blackheath/FPR_MEAP_ch1.pdf)
- [Reactive Web Applications with Play - Bernhardt](http://www.manning.com/bernhardt/) - [Chapter 1 (PDF)](http://www.manning.com/bernhardt/RWAwithPlay_MEAP_ch01.pdf)
- [Reactive Application Development - Devore](http://www.manning.com/devore/) - [Chapter 1 (PDF)](http://www.manning.com/devore/RAD_MEAP_ch1.pdf)
- [Functional and Reactive Domain Modeling - Ghosh](http://www.manning.com/ghosh2/) - [Chapter 1 (PDF)](http://www.manning.com/ghosh2/FRDM_MEAP_CH01.pdf)

## Video
- [An Introduction to Functional Reactive Programming](https://www.youtube.com/watch?v=ZOCCzDNsAtI)
- [Functional Reactive Programming in Elm - Evan Czaplicki](https://www.youtube.com/watch?v=JreO-Kl0Ed4)
- [Building Reactive Apps](https://www.youtube.com/watch?v=AFqFXlKrwRc) - [James Ward](http://www.jamesward.com/)

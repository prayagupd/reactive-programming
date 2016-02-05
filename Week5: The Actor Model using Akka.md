# Week 5: The Actor Model using Akka
> Actors are very lightweight concurrent entities. They process messages asynchronously using an event-driven receive loop. Pattern matching against messages is a convenient way to express an actor's behavior. They raise the abstraction level and make it much easier to write, test, understand and maintain concurrent and/or distributed systems. You focus on workflow—how the messages flow in the system—instead of low level primitives like threads, locks and socket IO.
-- <quote>[Akka.io](http://akka.io)</quote>

## Documentation
- [Akka.io](http://akka.io)

## Video
- [Youtube - Up, Up, and Out: Scaling Software with Akka](https://www.youtube.com/watch?v=GBvtE61Wrto)
- [Youtube - Up And Out Scaling Software With Akka - Jonas Bonér](https://www.youtube.com/watch?v=t4KxWDqGfcs)
- [Youtube - Akka 2.0: Scaling Up & Out With Actors](https://www.youtube.com/watch?v=3jbqTxstlC4)
- [Youtube - Above the Clouds: Introducing Akka](https://www.youtube.com/watch?v=UY3fuHebRMI)
- [Youtube - Deep Dive into the Typesafe Reactive Platform - Akka and Scala - with Nilanjan Raychaudhuri](https://www.youtube.com/watch?v=fMWzKEN6uTY)

## Slides
- [Jonas Boner - The Road to Akka Cluster and Beyond](http://www.slideshare.net/jboner/the-road-to-akka-cluster-and-beyond)

## Hint 1
The [Stash trait](http://doc.akka.io/docs/akka/snapshot/scala/actors.html#Stash) enables an actor to temporarily stash away messages that can not or should not be handled using the actor's current behavior. Upon changing the actor's message handler, i.e., right before invoking `context.become` or `context.unbecome`, all stashed messages can be "unstashed", thereby prepending them to the actor's mailbox. This way, the stashed messages can be processed in the same order as they have been received originally. To stash messages call `stash()`, to unstash call `unstashAll()`. Invoking `stash()` adds the current message (the message that the actor received last) to the actor's stash. It is typically invoked when handling the default case in the actor's message handler to stash messages that aren't handled by the other cases.

Use the `Stash` trait instead of the `Queue` in the example. 

## Hint 2 
To send messages, akka knows the following patterns, [tell](http://doc.akka.io/docs/akka/snapshot/scala/actors.html#Tell__Fire-forget), [ask](http://doc.akka.io/docs/akka/snapshot/scala/actors.html#Ask__Send-And-Receive-Future) and [forward](http://doc.akka.io/docs/akka/snapshot/scala/actors.html#Forward_message)
Only use `tell (fire and forget)`, you don't need the two other patterns. 
 
## Hint 3
The `BinaryTreeSet` class:
 
```scala
class BinaryTreeSet extends Actor with ActorLogging with Stash {
  import BinaryTreeNode._
  import BinaryTreeSet._

  var counter = 0

  def createRoot: ActorRef = {
    counter += 1
    context.actorOf(BinaryTreeNode.props(0, initiallyRemoved = true), "rt-" + counter)
  }

  var root = createRoot

  // optional
  def receive = normal

  // optional
  /** Accepts `Operation` and `GC` messages. */
  val normal: Receive = {
    case msg: Insert =>
      root ! msg

    case msg: Contains =>
      root ! msg

    case msg: Remove =>
      root ! msg

    case GC =>
      log.info("Receiving GC, creating newRoot and becoming garbageCollecting")
      val newRoot = createRoot
      root ! CopyTo(newRoot)
      context.become(garbageCollecting(newRoot))
  }

  // optional
  /** Handles messages while garbage collection is performed.
    * `newRoot` is the root of the new binary tree where we want to copy
    * all non-removed elements into.
    */
  def garbageCollecting(newRoot: ActorRef): Receive = LoggingReceive {
    case o: Operation =>
      log.info("(Enqueue): {}", o)
      stash()

    case CopyFinished =>
      log.info("(CopyFinished): destroying old tree nodes and replacing `root` with `newRoot` and replaying pending queue, and becoming normal")
      root ! PoisonPill
      root = newRoot
      unstashAll()
      context.become(normal)
  }
}
```


## Hint 4
The `BinaryTreeNode` class:

```scala
class BinaryTreeNode(val elem: Int, initiallyRemoved: Boolean) extends Actor with ActorLogging {
  import BinaryTreeNode._
  import BinaryTreeSet._

  // initiallyRemoved, is removed, will be GC'ed
  def node(value: Int) = context.actorOf(Props(new BinaryTreeNode(value, initiallyRemoved = false)), s"$value")

  var subtrees = Map[Position, ActorRef]()
  var removed = initiallyRemoved

  // optional
  def receive = normal

  def nodeInfo: String = s"elem: [$elem], initiallyRemoved: [$initiallyRemoved], removed: [$removed], subtrees: [$subtrees]"

  // optional
  /** Handles `Operation` messages and `CopyTo` requests. */
  val normal: Receive = LoggingReceive {
    // contains
    case msg @ Contains(requester, id, e) if e == elem && removed =>
      log.debug("(Contains): {}, returning ContainsResult({}, false)", nodeInfo,  id)
      requester ! ContainsResult(id, result = false)

    case msg @ Contains(requester, id, e) if e == elem && !removed =>
      log.debug("(Contains): {} == {}, not removed, returning ContainsResult({}, true)", e, elem, id)
      requester ! ContainsResult(id, result = true)

    case msg @ Contains(requester, id, e) if e < elem && subtrees.get(Left).isEmpty =>
      log.debug("(Contains): Left is empty: Subtrees: {}, returning ContainsResult({}, false)", subtrees, id)
      requester ! ContainsResult(id, result = false)

    case msg @ Contains(_, _, e) if e < elem && subtrees.get(Left).nonEmpty =>
      log.debug("(Contains): Left is nonEmpty, sending message: {}", msg)
      subtrees.get(Left).foreach (_ ! msg)

    case msg @ Contains(requester, id, e) if e > elem && subtrees.get(Right).isEmpty =>
      log.debug("(Contains): Right is empty: Subtrees: {}, returning ContainsResult({}, false)", subtrees, id)
      requester ! ContainsResult(id, result = false)

    case msg @ Contains(_, _, e) if e > elem && subtrees.get(Right).nonEmpty =>
      log.debug("(Contains): Right is nonEmpty, sending message: {}", msg)
      subtrees.get(Right).foreach (_ ! msg)

    // insert
    case msg @ Insert(requester, id, e) if e == elem =>
      log.debug("(Insert): {} == {}, returning OperationFinished({})", e, elem, id)
      removed = false
      requester ! OperationFinished(id)

    case msg @ Insert(_, _, e) if e < elem && subtrees.get(Left).isEmpty =>
      log.debug("(Insert): Left is empty, creating node and sending message: {}", msg)
      val leftNode = node(e)
      leftNode ! msg
      subtrees += Left -> leftNode

    case msg @ Insert(_, _, e) if e < elem && subtrees.get(Left).nonEmpty =>
      log.info("(Insert): Left is nonEmpty, sending message to node")
      subtrees.get(Left).foreach (_ ! msg)

    case msg @ Insert(_, _, e) if e > elem && subtrees.get(Right).isEmpty =>
      log.debug("(Insert): Right is empty, creating node and sending message: {}", msg)
      val rightNode = node(e)
      rightNode ! msg
      subtrees += Right -> rightNode

    case msg @ Insert(_, _, e) if e > elem && subtrees.get(Right).nonEmpty =>
      log.info("(Insert): Right is nonEmpty, sending message to node")
      subtrees.get(Right).foreach (_ ! msg)


    // Remove
    case msg @ Remove(requester, id, e) if e == elem =>
      log.debug("(Remove): {} == {}, returning OperationFinished({})", e, elem, id)
      removed = true
      requester ! OperationFinished(id)

    case msg @ Remove(requester, id, e) if e < elem && subtrees.get(Left).isEmpty =>
      log.info("(Remove): Left isEmpty, returning OperationFinished({})", id)
      requester ! OperationFinished(id)

    case msg @ Remove(_, _, e) if e < elem && subtrees.get(Left).nonEmpty =>
      log.info("(Remove): Left is nonEmpty, sending message: {}", msg)
      subtrees.get(Left).foreach (_ ! msg)

    case msg @ Remove(requester, id, e) if e > elem && subtrees.get(Right).isEmpty =>
      log.info("(Remove): Right isEmpty, returning OperationFinished({})", id)
      requester ! OperationFinished(id)

    case msg @ Remove(_, _, e) if e > elem && subtrees.get(Right).nonEmpty =>
      log.info("(Remove): Right is nonEmpty, sending message: {}", msg)
      subtrees.get(Right).foreach (_ ! msg)

    // CopyTo
    case CopyTo(_) if removed && subtrees.isEmpty =>
      log.info("(CopyTo): {}, node is removed and substrees are empty, becoming copied and sending OperationFinished to self", nodeInfo)
      context.become(copying(subtrees.values.toSet, insertConfirmed = false))
      self ! OperationFinished(elem)

    case CopyTo(treeNode) if !removed && subtrees.isEmpty =>
      log.info("(CopyTo): {}, node is not removed and substrees are empty, becoming copyTo and sending INSERT to newTree", nodeInfo)
      treeNode ! Insert(self, 1, elem)
      context.become(copying(subtrees.values.toSet, insertConfirmed = false))

    case CopyTo(treeNode) if removed && subtrees.isEmpty =>
      log.info("(CopyTo): {}, node is removed and substrees are empty, returning CopyFinished", nodeInfo)
      treeNode ! Insert(self, 1, elem)
      context.become(copying(subtrees.values.toSet, insertConfirmed = false))

    case msg @ CopyTo(treeNode) if removed && subtrees.nonEmpty =>
      val nodes: Set[ActorRef] = subtrees.values.toSet
      nodes.foreach (_ ! msg)
      context.become(copying(nodes, insertConfirmed = true))
      log.info("(CopyTo): {} node *is removed*, and subtrees is nonEmpty, becoming copying({}, true) ", nodeInfo, nodes)

    case msg @ CopyTo(treeNode) if !removed && subtrees.nonEmpty =>
      treeNode ! Insert(self, 1, elem)
      val nodes: Set[ActorRef] = subtrees.values.toSet
      nodes.foreach (_ ! msg)
      context.become(copying(nodes, insertConfirmed = false))
      log.info("(CopyTo): {} node is not removed, and subtrees is nonEmpty, becoming copying({}, false) ", nodeInfo, nodes)

    case msg =>
      log.info("Dropping msg: {}, info: {}", msg, nodeInfo)
  }

  // optional
  /** `expected` is the set of ActorRefs whose replies we are waiting for,
    * `insertConfirmed` tracks whether the copy of this node to the new tree has been confirmed.
    */
  def copying(expected: Set[ActorRef], insertConfirmed: Boolean): Receive = LoggingReceive {
    case OperationFinished(_) if expected.isEmpty =>
      log.info("(OperationFinished): {}, I've been copied, children are finished, returning CopyFinished to parent and stopping self", nodeInfo)
      context.parent ! CopyFinished
      context.stop(self)

    case OperationFinished(_) if expected.nonEmpty =>
      log.info("(OperationFinished): {}, I've been copied, waiting on children, becoming copying({}, true)", nodeInfo)
      context.become(copying(expected, insertConfirmed = true))

    case CopyFinished if expected.isEmpty && insertConfirmed =>
      log.info("(CopyFinished): child finished, no more children and I've been copied, returning CopyFinished to parent and stopping self")
      context.parent ! CopyFinished
      context.stop(self)

    case CopyFinished if expected.nonEmpty =>
      val newExpected = expected.filterNot(_ == sender())
      context.become(copying(newExpected, insertConfirmed))
      if(newExpected.isEmpty) {
        log.info("(CopyFinished): all children are finished, returning CopyFinished to parent and stopping self")
        context.parent ! CopyFinished
        context.stop(self)
      } else {
        log.info("(CopyFinished): child finished, waiting for another child, becoming copying({}, {})", newExpected, insertConfirmed)
      }

    case CopyFinished if expected.isEmpty && !insertConfirmed =>
      log.info("(CopyFinished): children are finished, but I'm not copied yet, becoming copying({}, {})", expected, insertConfirmed)
      context.become(copying(expected, insertConfirmed))
  }
}
```

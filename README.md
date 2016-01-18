PayPal Scala Style Guidelines
=================

This repository contains style guidelines for writing Scala code at PayPal. Here are our goals for style
guides in this repository:

1. Our style guidelines should be clear.
2. We all should agree on our style guidelines.
3. We should change this guideline as we learn and as Scala evolves.
4. Keep the [scalastyle-config.xml](https://github.com/paypal/scala-style-guide/blob/develop/scalastyle-config.xml) in our projects in line with this guide as much as possible.

This repository follows git-flow. If you have a new style guideline, open a pull request against `develop`. We'll
discuss in the PR comments. The official guidelines live in [master](https://github.com/paypal/scala-style-guide/blob/master/README.md)

# Whitespace

* Use two spaces for indentation.
* Add a newline to the end of every file.

# Line Length

* Use a maximum line length of 160 characters.

# Names

## Functions, Classes and Variables

Use camel case for all function, class and variable names. Classes start with an upper case letter, and values and
functions start with a lower case. Here are some examples:

```scala
class MyClass {
  val myValue = 1
  def myFunction: Int = myValue
}
```

## Package Objects

File names for package objects must match the most specific name in the package.
For example, for the `com.paypal.mypackage` package object, the file
should be named `mypackage.scala` and the file should be structured like this:


```scala
package com.paypal

package object mypackage {
  //...
}
```


# Throwables

1. Prefer non-case classes when defining exceptions and errors unless you plan on pattern matching on the exception.
2. If providing a cause is desirable and not mandatory then define it as an option. Note, the use of ```.orNull```.

```scala
class CrazyException(msg: String, cause: Option[Throwable] = None) extends Exception(msg, cause.orNull)
class SuperCrazyError(msg: String, cause: Option[Throwable] = None) extends Error(msg, cause.orNull)
```

# Imports

## Ordering

IntelliJ's code style configuration allows for automatic grouping of imports by namespace.
We use the following ordering, which you should add to your IntelliJ configuration
(found in Preferences > Code Style > Scala):

```
java
___blank line___
scala
___blank line___
akka
spray
all other imports
___blank line___
com.ebay
com.paypal
```

You should also regularly run IntelliJ's Optimize Imports (Edit > Optimize Imports) against
your code before merging with `develop`, to maintain import cleanliness.

## Location

Most `import`s in a file should go at the top, right below the package name. The
only time you should break that rule is if you import some or all names from
an `object` from inside a `class`. For example:

```scala
package mypackage

import a.b.c.d
import d.c.b.a

class MyClass {
  import Utils._
  import MyClassCompanion.A
}

object MyClassCompanion {
  case class A()
}

object Utils {
  case class Util1()
  case class Util2()
}
```

## Predef

Never `import` anything inside of Scala's
['Predef'](http://www.scala-lang.org/api/2.10.3/index.html#scala.Predef$)
object. It is automatically imported for you, so it's redundant to manually
`import`. IntelliJ will often try to import it, so be aware.

# Values

Use `val` by default. `var`s should be limited to local variables or `private` class variables. Their usage
should be well documented, including any considerations that developers must make with respect to concurrency.

One common usage of `var`s will be inside of Akka `Actor`s. Here's an example of that usage:

```scala
class MyActor extends Actor {
  //This is local state to the actor that represents the current sum of all integers it has received.
  //It must be marked private and never accessed outside of the processing of a single message, to ensure
  //that it's concurrency safe. Additionally, don't acquire a lock before you access this variable.
  //Doing so might block message processing unnecessarily. If you follow all of these rules related to concurrency,
  //you shouldn't need a lock anyway.
  private var sum: Int = 0

  override def receive: Receive {
    case i: Int => sum = sum + 1
  }
}
```

## Modifiers

Optional variable modifiers should be declared in the following order: `override access (private|protected) final implicit lazy`.

For example,

```scala
private implicit lazy val someSetting = ...
```

# Functions

Rules to follow for all functions:

* Always put a space after `:` characters in function signatures.
* Always put a space after `,` in function signatures.

## Public Functions
All public functions and methods, including those inside `object`s must have:

* A return type
* Scaladoc including a function overview, information on parameters and information on the return value

Here is a complete example of a function inside of an `object`.

```scala
object MyObject {
  /**
   * returns the static integer 123
   * @return the number 123
   */
  def myFunction: Int = {
    123
  }
}
```

## Anonymous functions

Anonymous functions start on the same line as preceding code. Declarations
start with ` { ` (note the space before and after the `{`). Arguments are then
listed on the same line. A few more notes:

* Do not use braces after the argument list, just start the function body on the next line.
* Argument types are not necessary. You should use them if it makes the code clearer though.

Here's a complete example example:

```scala
Option(123).map { number =>
  println(s"the number plus one is: ${number + 1}")
}
```

Anonymous functions that contain a single binary operation or method call on the input can use the underscore syntax and should be surrounded by parentheses. For example:

```scala
val list = List("list", "of", "strings")
list.filter(_.length > 2)
list.filter(_.contains("i"))
```

## Passing named functions

If the function takes a single argument, then arguments and underscores should be omitted.

For example:
```scala
Option(123).map(println)
```

is preferred over

```scala
Option(123).map(println(_))
```

## Calling functions

Prefer dot notation except for these specific scenarios:
1. Specs2 matchers
2. Akka ```pipeTo```, ```!```, ```?```

For example:
```scala
Option(someInt).map(println)
```

is preferred over

```scala
Option(someInt) map println
```

##### Rule Exceptions

Java <=> Scala interoperation on primitives: Scala and Java booleans, for example, are not directly compatible, and
require an implicit conversion between one another. Normally this is handled automagically by the Scala compiler, but
the following code would reveal a compiler error:

```scala
Option(true).foreach(someMethodThatTakesAJavaBoolean)
```

A named parameter is *not* necessary to achieve this. An underscore should be used to resolve the implicit conversion.
To avoid confusion, please also add a note that a Scala <=> Java conversion is taking place:

```scala
Option(true).foreach(someMethodThatTakesAJavaBoolean(_))  // lol java
```

# Logic Flows

In general, logic that handles a choice between two or more outcomes should prefer to use `match`.

## Match Statements

When you `match` on any type, follow these rules:

1. indent all `case` statements at the same level, and put the `=>` one space to
  the right of the closing `)`
2. if your expression is one line:
  1. put it on the same line as the `case` if doing so will not make the line
    excessively long
  2. put it on the line below the `case`, indented one level in from the `case`
3. If your expression is more than one line, put it on the line below the case,
  indented one level in from the `case`.
4. Do not add extra newlines in between each case statement.
5. Filters on case statements should be on the same level if doing so will not make
  the line excessively long.

Here's a complete example:

```scala
Option(123) match {
  case Some(i: Int) if i > 0 =>
    val intermediate = doWorkOn(i + 1)
    doMoreWorkOn(intermediate)
  case _ => 123
}
```

## Option

Flows with `Option` values should be constructed using the `match` keyword as follows.

```scala
def stuff(i: Int): Int = { ... }

Option(123) match {
  case None => 0
  case Some(number) => stuff(number)
}
```

The `.fold` operator should generally not be used. Simple, single-line patterns are acceptable for `.fold`, such as:

```scala
def stuff(i: Int): Int = { ... }

Option(123).fold(0)(stuff)
```

Similarly, simple patterns are acceptable for `.map` with `.getOrElse`, such as:

```scala
def stuff(i: Int): Int = { ... }

Option(123).map(stuff).getOrElse(0)
```

You should enforce expected type signatures, as `match` does not guarantee consistent types between match outcomes
(e.g. the `None` case could return `Int`, while the `Some` case could return `String`).

## For Comprehension

For comprehensions should generally not be wrapped in parentheses in order to recover, flatMap, etc.
Instead, separate the for comprehension into its own variable and perform additional operations on that.

```scala
val userAddress = for {
  address <- loadAddress()
} yield address

userAddress.recover {
  // recover from loadAddress exception
}.flatMap {
  // pull info from address, etc.
}
```

In addition, if performing additional operations on the result of the yield as part of the result of the for comprehension itself,
code should be separated by braces and begin on a newline. For example:

```scala
for {
  address <- loadAddress() // returns Option
} yield {
  address match {
    case Some(s) => ...
    case None => ...
  }
}
```

## Case Ordering

As a general practice, the ordering of case statements should be prioritized from most likely to be the executed to least likely.
For example, if you expect a `Try` to generally succeed, the first case in the match should be `Success(s)`, followed by `Failure(f)`.

The same goes for if/else statements, naturally.

# Akka

## Ask and Tell

Prefer ```!``` and ```?``` over ```.tell``` and ```.ask```.

```scala
someActor1 ! msg
someActor2 ? msg
```

is preferred over

```scala
someActor1.tell(msg)
someActor2.ask(msg)
```

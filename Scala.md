# Scala

## Type-safe builders

Add a generic type parameter for each value parameter without a default value.
Then ask for an implicit evidence, Scala automatically narrows types in `build`
method:
```scala
case class Builder[A, B](a: A = null, b: B = null) {
  def setA(a: Int): Builder[Int, B] = copy(a = a)
  def setB(b: String): Builder[A, String] = copy(b = b)
  def build(using
    evidenceA: A =:= Int,
    evidenceB: B =:= String) = ???
}
```

Less pedantic implementation can even omit setter methods, because type
inference alters type parameters on the fly for each `copy` call.

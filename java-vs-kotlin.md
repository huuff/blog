## The indispensables
(This is true as of Java 17 and Kotlin 1.7)

### Named and default arguments
In Kotlin, functions parameters can have defaults so they can be omitted at the call site:

```kotlin
fun makePizza(baseType: String = "margherita"): Pizza {
  ...
}

// This argument is optional, you can provide it or leave it out and it'll take the default value
val margherita = makePizza()
val hawaiian = makePizza("hawaiian")
```

This might present other issues, for example, what would you do if two arguments with defaults have the same type?

```kotlin
fun makePizza(leftHalfBase: String = "margherita", rightHalfBase: String = "margherita"): Pizza {
  ...
}

// What if you want to only specify a type for the right half? This won't cut it, as the first argument is the left half...
val margheritaHawaiian = makePizza("hawaiian")
// You could provide a first value, but that negates the benefit of having a default!
val anotherMargheritaHawaiian = makePizza("margherita", "hawaiian")
```

Fortunately, Kotlin also provides *named arguments*, which allow us to pass arguments depending on their names instead of their position:

```kotlin
makePizza(rightHalfBase = "hawaiian")
```

#### The benefits
* Default arguments allow us to add new parameters to a function without changing existing code, as long as that new argument has a default value (not unlike schema migration for databases)

```kotlin
// Before adding a parameter
fun makePizza(baseType: String = "margherita")

// After adding a parameter
fun makePizza(baseType: String, size: "medium")

// No calls at all must be changed, neither in our project nor in any one that uses ours as a dependency.
```

* Default arguments can be used to provide sensible defaults, allowing a simpler, cleaner interface for experimenting while also allowing us to customize the call to suit our needs more.
* Named arguments make code much more explicit. A classic issue is that arguments that don't map well to a positional analogy, are prone to be misunderstood (with the consequence of potential bugs if the types of the arguments and they can't be checked at compile-time)

```java
// Stop me if you've heard this one before...
assertEquals(arg1, arg2);
// Which is the expected and which is the actual?
assertEquals(actual, expected)!;
// Wrong! You'll get a puzzling error message along the lines of...
// Expected actual, got expected ==TODO: improve this==
```

```kotlin
// In kotlin, all is clean and order doesn't matter anymoe
assertEquals(expected = expected, actual = actual)
```


#### Java alternatives
##### Java alternative: Telescoping methods
The canonical proposed solution to this is using *telescoping methods*, which basically consists of writing a method overload for every possible combination of arguments and default arguments:

```java

public Pizza makePizza(String leftHalfBase, String rightHalfBase) { ... }
public Pizza makePizza(String leftHalfBase) {
  return makePizza(leftHalfBase, "margherita");
}
public Pizza makePizza() {
  return makePizza("margherita", "margherita");
}

```

This can get unwieldly pretty fast. Even with just a couple parameters more the amount of overloads can quickly ramp up. Other downsides are:
* The signal-to-noise ratio decreases, a pretty usual property in Java programs. Several of your methods are just small overloaded variations of a base method, which imposes an additional mental tax on the reader of the code and buries down methods with useful logic.
* This doesn't even solve our second problem! There is no way to provide the second parameter without providing the first one.
* Without named arguments, it's hard for the caller to know which argument they're passing for which parameter if they have the same type. In this case, the positional nature of the parameters (left, right) gives it away, but for many other cases it isn't that clear, and it's prone to errors.

##### Java alternative: Builder pattern
A classic Java argument ^1 is that your methods shouldn't have long parameters lists. Long parameters lists indicate that we should refactor them into an object. A nice solution to this is the *builder pattern*, where a mutable object's state is modified with a chain of operations that specify how we want to build the resulting objet. Also, if we add a *fluent interface* where methods can be chained on the same call, the result is quite stylish:

```Java
val margheritaHawaiian = new PizzaBuilder()
  .leftHalf("hawaiian")
  .rightHalf("margherita")
  .toppings(List.of("pepperoni", "onion"))
  .size("large")
  .build();
```


## References
* *Clean Code*, by Robert C. Martin

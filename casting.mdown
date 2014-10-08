#Casting

##What problem are we solving?

Often times, we are provided with an object that is typed `id` (effectively, unknown type.) Classic examples: The contents of NSArray and NSDictionary are not known to the compiler. In such cases, the compiler / linter will complain if we attempt to use object-specific methods, convenience initializers, or object literals, even if the underlying object is of the appropriate type to use those tools. How might we stop the compiler / linter from complaining?

Other times we are provided with an object that has methods that override its superclass's methods. But what if we want access to the superclass's method implementation?


##What is casting?

Casting is the process of transforming one object type into another object type. Casting communicates to the compiler and linter that we would like to have access to features such as an object's methods and literal syntax if applicable and that we explicitly know that it will be okay to use them and will not crash the program. But words of warning: Casting is a quick way to runtime errors given we are effectively telling the compiler and linter that "we know better." That means our app may crash when in use if casting is poorly applied. Be extra thoughtful when casting!

##How to cast.

The syntax for casting is simple. Assume for this example that we have a project with two classes, Vehicle and Porsche. Porsche is a subclass of the Vehicle class.


```
Vehicle *newCar = [Vehicle alloc] init];
Porsche *new911Model = (Porsche *)newCar;
```

In the above code snippet, we have initialized a new Vehicle, and cast it as a Porsche by adding the `(Porsche *)` in front of the Vehicle object. Casting a general object to be a more specific type of object is called downcasting. As far as the compiler is concerned now, we can use the new911Model as a Porsche, even though it is actually only a Vehicle. That does not mean, however, that the behavior of new911Model will be exactly the same as it would be if we had initialized a Porsche. So, here are the major distinctions in behavior:


- When the same method is defined in both parent class and child class, downcasting an object will result in the parent's method implementation being called.


- On the contrary, when upcasting, if the same method is defined in both parent class and child class, upcasting will result in the child's method being called.


- In both situations, you can expect that methods in the child class (and only in the child class) will NOT run. Either, they will cause an error in the compiler in the case of downcasting, or they will cause an error at runtime, as in the case of upcasting.

It is also possible to cast in-line by putting parentheses around an object. For instance, if we had a Car object and a Porsche object, and the Porsche object had a property called isTurboCharged. The following would work:

```
FISCar *newCar = [FISPorsche alloc] init];
((Porsche *)newCar).isTurboCharged = YES;
```


## Unexpected behaviors

### Integer Division

A fairly common situation is the division of two integers that result in a floating point number. Take the following example:
```
CGFloat result = 9/4;
```
If you were to write the above formula, `result` would evaluate to the number 2.

In order to get 2.25, you would first have to cast the parameters `9` and `4` like so:

```
CGFloat result = (CGFloat)9/CGFloat(4);
```


## Common error messages


Let's say we attempt to call `engageWarpDrive` on any old `FISCar` as opposed to our pimped out FISTrekkyCar.

```
[FISCar engageWarpDrive];
```

Given only an FISTrekkyCar (a subclass of FISCar) has the method `engageWarpDrive`, we can expect to see the following error in our debugger:

```
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[FISCar engageWarpDrive]: unrecognized selector sent to instance 0x7fde0be0fd10'
```

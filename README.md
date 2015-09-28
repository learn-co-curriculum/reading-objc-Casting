# Casting

## What Problem Does Casting Solve?

Often times, we are provided with an object that is typed `id` (effectively, unknown type.) Classic examples: The contents of `NSArray` and `NSDictionary` are not known to the compiler. In such cases, the compiler will complain if we attempt to use object-specific methods, convenience initializers, or object literals, even if the underlying object is of the appropriate type to use those tools. How might we stop the compiler from complaining?

Other times we are provided with an object that has methods that override its superclass's methods. But what if we want access to the superclass's method implementation?


## What Is Casting?

Casting is the process of transforming one object type into another object type. Casting communicates to the compiler and linter that we would like to have access to features such as an object's methods and literal syntax if applicable and that we explicitly know that it will be okay to use them and will not crash the program. But words of warning: Casting is a quick way to runtime errors given that we are effectively telling the compiler  that "we know better." That means our app may crash when in use if casting is poorly applied. Be extra thoughtful when casting!

## How To Cast

The syntax for casting is simple. Assume for this example that we have a project with two classes, `FISVehicle` and `FISPorsche`. `FISPorsche` is a subclass of the `FISVehicle` class.

```objc
FISVehicle *newCar = [FISVehicle alloc] init];
FISPorsche *new911Model = (FISPorsche *)newCar;
```

In the above code snippet, we have initialized a new `FISVehicle`, and cast it as a `FISPorsche` by adding `(FISPorsche *)` in front of the `FISVehicle` object. Casting a general object to be a more specific type of object is called downcasting. As far as the compiler is concerned, we can use the `new911Model` as a `FISPorsche`, even though it is actually only a `FISVehicle`. That does not mean, however, that the behavior of `new911Model` will be exactly the same as it would be if we had initialized a `FISPorsche`. So, here are the major distinctions in behavior:


- When the same method is defined in both parent class and the child class, downcasting an object will result in the parent's method implementation being called.


- On the contrary, when upcasting, if the same method is defined in both the parent class and the child class, upcasting will result in the child's method being called.


- In both situations, you can expect that methods in the child class (and only in the child class) will NOT run. Either, they will cause an error in the compiler in the case of downcasting, or they will cause an error at runtime, as in the case of upcasting.

It is also possible to cast in-line by putting parentheses around an object. For instance, if we had a `FISCar` object and a `FISPorsche` object, and the `FISPorsche` object had a property called `isTurboCharged`. The following would work:

```objc
FISCar *newCar = [FISPorsche alloc] init];
((FISPorsche *)newCar).isTurboCharged = YES;
```


## Unexpected behaviors

### Integer Division

A fairly common situation is the division of two integers that result in a floating point number. Take the following example:

```objc
CGFloat result = 9 / 4;
```
If we were to write the above formula, `result` would evaluate to the integer value `2`, truncating the decimal value. This case is solved easily enough by writing one of the operands as a float value:

```objc
CGFloat result = 9 / 4.0;
```

The `result` variable will now hold the decimal value `2.25`.

However, what if we wanted the decimal result of a division operation that uses two integer *variables* as its operands?

```objc
NSInteger a = 9;
NSInteger b = 4;

CGFloat result = a / b;
```

Assuming that the variables `a` and `b` are sourced from somewhere else in the code and cannot be directly modified to `CGFloat`s, we could instead cast one or both of the variables as `CGFloat`s in order to get a decimal result of the division operation. This could be written as:

```objc
NSInteger a = 9;
NSInteger b = 4;

CGFloat result = (CGFloat)a / (CGFloat)b;
```

The `result` variable will now successfully capture `2.25`.

**Note:** *Casting floats values or other primitives does not employ the* `*` *character.*


## Common Error Messages


Let's say we have another subclass of `FISCar` called `FISTrekkyCar` that has a new method called `engageWarpDrive`. What happens if we downcast a `FISCar` to a `FISTrekkyCar` and attempt to call `engageWarpDrive` on it?

```objc
FISTrekkyCar *enterprise = (FISTrekkyCar *)[[FISCar alloc] init];

[enterprise engageWarpDrive];
```

Since the `enterprise` object was initialized as a `FISCar` and is not actually a `FISTrekkyCar` which is that class that can receive a call of the `engageWarpDrive` method, the following error will print in the debug console: 

```
*** Terminating app due to uncaught exception 'NSInvalidArgumentException',
 reason: '-[FISCar engageWarpDrive]: unrecognized selector sent to instance 
 0x7fde0be0fd10'
```

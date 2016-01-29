# Casting

## Objectives

1. Learn the purpose and forms of casting values.
2. Recognize the different use-cases for casting primitives and objects.

## What Problem Does Casting Solve?

Often times, we are provided with a value that is not *quite* the type we want. Usually, this takes the form of an object that is typed `id` (effectively "unknown object type"). The classic example is methods that access elements of arrays and dictionaries. The contents of `NSArray` and `NSDictionary` are not known to the compiler, so many methods that manipulate collections have no choice but to operate with the `id` type. As we've seen, that situation is a bit tricky, since the compiler will just trust that you know what you're doing when calling methods on something with type `id`. For example:

```objc
NSDictionary *userInfo = @{
    @"name": @"Tim",
    @"age": @12,
    @"favoriteColors": @[ @"red", @"green", @"blue" ]
};

id age = userInfo[@"age"];
NSLog(@"%@", [age uppercaseString]);
``` 

Xcode just assumes that we actually wanted to call `uppercaseString` on `age`, and will let this compile, even though `age` is actually an `NSNumber`. This will result in a crash at runtime -- good old `unrecognized selector sent to instance`.

There is yet another annoyance with `id`: the compiler disallows dot-notation on variables with that type. Check it out:

```objc
id name = userInfo[@"name"];
NSLog(@"%@", name.uppercaseString);

// or...

NSLog(@"%@", userInfo[@"name"].uppercaseString);
```

Both of these result in the same error: `Property 'uppercaseString' not found on object of type 'id'`. Ugh.


So, what's a budding young programmer to do? How do we force the compiler to interpret a value as a different type? Cast!


## What Is Casting?

**Casting** is the act of forcing the compiler to interpret a value as a specific type. For instance, we could *cast* `age` in the previous code to an `NSNumber *` so that we can use dot-syntax and get the usual compiler niceties (such as a compiler error when we try to do the impossible, like capitalize an `NSNumber`!).

Casting lets us fill the compiler in on details it couldn't possibly know. But, a word of warning: casting is a quick way to get runtime errors if we happen to mess up. **Casts between types almost *always* compile**, even if they're nonsensical. So it's perfectly valid to cast, say, an `NSNumber` to an `NSString`, only to crash when you try to run string methods on the `NSNumber`.

**Don't confuse casting objects with converting between types!** Casting doesn't actually change the type of the object being cast, it just asks the compiler to treat the value as a different type. For example, casting an `NSNumber` to an `NSString` doesn't get you the string version of the number, it just gets you a string variable that actually holds a number. That's a recipe for a crash.


## How To Cast

There are two types of casts: implicit and explicit. **Implicit casts** happen whenever you assign a value to a different type of variable:

```objc
NSString *name = userInfo[@"name"];
```

Even though the dictionary access will return a value of type `id`, this is a valid statement. The `id` is implicitly cast to an `NSString` and assigned to the variable `name`. We are now free to use dot syntax or anything else we're used to with an `NSString *` variable. This type of casting only works when the value being cast is of type `id` or the compiler can guarantee that the cast makes sense.

**Explicit casts** are needed when the value being cast is not of type `id`, or when the compiler can't guarantee that you're doing something sane. Assume for this example that we have two classes, `Vehicle` and `Boat`. `Boat` is a subclass of the `Vehicle` class. If we have a variable of type `Vehicle`, but we are *sure* it's actually a `Boat` (maybe we called `-isKindOfClass:` to check), we can have the compiler reinterpret the value like this:

```objc
Vehicle *vehicleThatIsActuallyABoat = ...;  // Maybe this was an argument
// [vehicleThatIsActuallyABoat dropAnchor];  // This would be an error, since at this point the compiler only knows that this is a Vehicle, which does't have a method to drop an anchor

Boat *boat = (Boat *)vehicleThatIsActuallyABoat;
[boat dropAnchor];  // That's better! Boats can actually do that.
```

In the above code snippet, we have asked the compiler to treat `vehicleThatIsActuallyABoat` as a `Boat` by adding the `(Boat *)` in front of the `Vehicle` value. That lets us call methods that belong exclusively to boats, like `dropAnchor`.

Remember that no real work actually happened! The cast effectively fills the compiler in on the fact that we have a `Boat` and that it should let us call `Boat`-specific stuff on the value. `vehicleThatIsActuallyABoat` and `boat` are the *exact same object*.


It is also possible to cast in-line by putting parentheses around the whole expression. For instance:

```objc
[((Boat *)vehicleThatIsActuallyABoat) dropAnchor];
```


### Downcasting and upcasting

The cast we just saw, from a `Vehicle` to a more specific subclass of it (namely `Boat`) is a **downcast**. It is a cast *down* the tree of inheritance.

We can also cast upwards, toward the superclass. This is (surprise!) an **upcast**. For example:

```objc
Vehicle *vehicleThatIsActuallyABoat = (Vehicle *)boat;

// or, since the compiler knows this will work, we can do this implicitly:

Vehicle *vehicleThatIsActuallyABoat = boat;
```

Yet again, no real work actually happened! The upcast just causes the compiler to reinterpret the `Boat` as a `Vehicle`, thus cutting us off from `Boat`-specific stuff on the value. `vehicleThatIsActuallyABoat` and `boat` are still the *exact same object*. Calling methods on `vehicleThatIsActuallyABoat` that are also in `Vehicle` will result in the boat implementations being executed:

```objc
BOOL waterVehicle = vehicleThatIsActuallyABoat.canTravelOnWater;  // This will be YES (i.e., Boat's implementation) even though we called it on a thing that is typed as a Vehicle.
```


## Casting Primitives

All of the examples we've seen so far have been casts between object types. We can also cast primitive values (`NSInteger`, `BOOL`, etc.) to convert their types. **Unlike casts between object types, casts between primitives actually convert values!** For instance, casting a `CGFloat` to an `NSInteger` truncates the value (i.e., it removes the decimal part):

```objc
CGFloat pi = 3.14159;
NSInteger piInIndiana = (NSInteger)pi;  // This is 3. See https://en.wikipedia.org/wiki/Indiana_Pi_Bill
``` 

### Integer Division

A fairly common use case for primitive casting arises when dividing two integers. This operation (e.g. `9 / 4`) is defined to return another integer, *not* a floating-point number. This oddity can cause issues if you're not expecting it. Take the following example:

```objc
CGFloat result = 9/4;
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

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/reading-objc-Casting' title='Casting'>Casting</a> on Learn.co and start learning to code for free.</p>

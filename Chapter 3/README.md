## Iteration
The iterator pattern has been around for decades,andsuggests a “standardized” approach to consuming data from a source one *chunk* at a time. The idea is that it’s more common and helpful to iterate the data source—to progressively handle the collection of data by processing the first part, then the next,and so on, rather than handling the entire set all at once.

The iterator pattern defines a data structure called an “iterator” that has a reference to an underlying data source (like the query result rows), which exposes a method like *next().* Calling *next()* returns the next piece of data (i.e., a “record” or “row” from a database query).

The importance of the iterator pattern is in adhering to a *standard* way of processing data iteratively, which creates cleaner and easier to understand code, as opposed to having every data structure/source define its own custom way of handling its data.

## Consuming Iterators
With the ES6 iteration protocol in place, it’s workable to consume a data source one value at a time, checking after each *next()* call for *done* to be *true* to stop the iteration.But this approach is rather manual, so ES6 also included several mechanisms(syntax and APIs)for standardized consumption of these iterators.

One such mechanism is the *for..of* loop:
```Js
// given an iterator of some data source:
var it=/* .. */;
// loop over its results one at a time
for(let val of it) {
    console.log(`Iterator value:${val}`);
}
// Iterator value: ..
// Iterator value: ..
// ..
```

Another mechanism that’s often used for consuming iterators is the *...* operator. This operator actually has two symmetrical forms:*spread* and *rest*(or *gather*, as I prefer). The *spread* form is an iterator-consumer.

To *spread* an iterator,you have to have *something* to spread it into.There are two possibilities in JS:an array or an argument list for a function call.

An array spread:
```Js
// spread an iterator into an array,
// with each iterated value occupying
// an array element position.
var vals = [ ...it ];
```

A function call spread:
```Js
// spread an iterator into a function,
// call with each iterated value
// occupying an argument position.
doSomethingUseful( ...it );
```
## Iterables
The iterator-consumption protocol is technically defined for consuming *iterables;* an iterable is a value that can be iterated over.

The protocol automatically creates an iterator instance froman iterable, and consumes *just that iterator instance* to its completion. This means a single iterable could be consumed more than once; each time, a new iterator instance would becreated and used.

ES6 defined the basic data structure/collection types in JS as iterables. This includes strings, arrays, maps, sets, and others.

Since arrays are iterables,we can shallow-copy an array using iterator consumption via the *...* spread operator:
```Js
var arrCopy=[ ...arr ];
```

A Map data structure uses objects as keys, associating a value(of any type) with that object. Maps have a different default iteration than seen here, in that the iteration is not just over the map’s values but instead its *entries.* An *entry* is a tuple(2-element array) including both a key and a value.

For the most part,all built-in iterables in JS have three iterator forms available:keys-only(*keys()*),values-only(*values()*),and entries (*entries()*).

## Closure
Closure is when a function remembers and continues to access variables from outside its scope, even when the function is executed in a different scope.

We see two definitional characteristics here. First, closure is part of the nature of a function. Objects don’t get closures,functions do. Second, to observe a closure, you must execute a function in a different scope than where that function was originally defined.

Consider:

```Js
function greeting(msg) {
    return function who(name) {
        console.log(`${msg},${name}!`);
    };
}
var hello = greeting("Hello");
var howdy = greeting("Howdy");
hello("Kyle");// Hello, Kyle!
hello("Sarah");// Hello, Sarah!
howdy("Grant");// Howdy, Grant!
```

First, the *greeting(..)* outer function is executed, creatingan instance of the inner function *who(..);* that function closes over the variable *msg,* which is the parameter from the outer scope of *greeting(..)*. When that inner function is returned, its reference is assigned to the *hello* variable in the outer scope. Then we call *greeting(..)* a second time,creating a new inner function instance, with a new closure over a new *msg*, and return that reference to be assigned to *howdy*.

When the *greeting(..)* function finishes running,normally we would expect all of its variables to be garbage collected(removed from memory). We’d expect each *msg* to go away,but they don’t. The reason is closure. Since the inner function instances are still alive (assigned to *hello* and *howdy,* respectively), their closures are still preserving the *msg* variables.

These closures are not a snapshot of the *msg* variable’s value; they are a direct link and preservation of the variable itself. That means closure can actually observe (or make!) updates to these variables over time.

```Js
function counter(step = 1) {
    var count = 0;
    return function increaseCount(){
        count = count + step;
        return count;
    };
}
var incBy1 = counter(1);
var incBy3 = counter(3);
incBy1();// 1
incBy1();// 2
incBy3();// 3
incBy3();// 6
incBy3();// 9
```

Each instance of the inner *increaseCount()* function isclosed over both the *count* and *step* variables from its outer *counter(..)* function’s scope. *step* remains the same overtime, but *count* is updated on each invocation of that inner function. Since closure is over the variables and not just snapshots of the values, these updates are preserved.

## *this* Keyword

As discussed previously, when a function is defined, it is *attached* to its enclosing scope via closure. Scope is the set of rules that controls how references to variables are resolved.

But functions also have another characteristic besides their scope that influences what they can access.This characteristic is best described as an *execution context*, and it’s exposed to the function via its *this* keyword.

Scope is static and contains a fixed set of variables available at the moment and location you define a function, but a function’s execution *context* is dynamic, entirely dependent on **how it is called**(regardless of where it is defined or even called from).

```Js
function classroom(teacher) {
    return function study() {
        console.log(`${teacher}says to study${this.topic}`);
        };
    }
    var assignment = classroom("Kyle");
```
The outer *classroom(..)* function makes no reference to a *this* keyword, so it’s just like any other function we’ve seen so far. But the inner *study()* function does reference *this*,which makes it a *this*-aware function. In other words, it’s a function that is dependent on its *execution context.*

The inner *study()* function returned by *classroom("Kyle")* is  assigned to a variable called *assignment.* So how can *assignment()* (aka *study()*) be called?

```Js
assignment();
// Kyle says to study undefined  -- Oops :(
```

In this snippet, we call *assignment()* as a plain, normal function, without providing it any *execution context*.

Since this program is not in strict mode, context-aware functions that are called **without any context specified** default the context to the global object (*window* in the browser). As there is no global variable named *topic* (and thus no such property on the global object),*this.topic* resolves to undefined.

Now consider:

```Js
var homework = {
    topic:"JS",
    assignment:assignment
};
homework.assignment();
// Kyle says to study JS
```
Lastly:
```Js
var otherHomework = {
    topic:"Math"
};
assignment.call(otherHomework);
// Kyle says to study Math
```
A third way to invoke a function is with the *call(..)* method, which takes an object (*otherHomework* here) to use for setting the *this* reference for the function call. The property reference *this.topic* resolves to "Math".

## Prototypes

Where *this* is a characteristic of function execution, a prototype is a characteristic of an object,and specifically resolution of a property access.

Think about a prototype as a linkage between two objects; the linkage is hidden behind the scenes, though there are ways to expose and observe it. This prototype linkage occurs when an object is created; it’s linked to another object that already exists.

A series of objects linked together via prototypes is called the“prototype chain.”

The purpose of this prototype linkage (i.e., from an object B to another object A) is so that accesses against B for properties/methods that B does not have, are *delegated* to A to handle. Delegation of property/method access allows two(or more!) objects to cooperate with each other to perform atask.

```Js
var homework = {
    topic:"JS"
};
homework.toString();// [object Object]
```
*homework.toString()* works even though *homework* doesn’t have a *toString()* method defined; the delegation invokes *Object.prototype.toString()* instead.

## Object Linkage
To define an object prototype linkage, you can create the object using the *Object.create(..)* utility:

```Js
var homework = {
    topic:"JS"
};
var otherHomework = Object.create(homework);
otherHomework.topic;// "JS"
```

The first argument to *Object.create(..)* specifies an object to link the newly created object to, and then returns the newly created (and linked!) object.

Delegation through the prototype chain only applies for accesses to lookup the value in a property. If you assign to aproperty of an object, that will apply directly to the objectregardless of where that object is prototype linked to.

```Js
homework.topic;
// "JS"
otherHomework.topic;
// "JS"
otherHomework.topic="Math";
otherHomework.topic;
// "Math"
homework.topic;
// "JS" -- not "Math"
```
The assignment to *topic* creates a property of that name directly on *otherHomework*; there’s no effect on the *topic* property on *homework*. The next statement then accesses *otherHomework.topic*, and we see the non-delegated answer from that new property: "Math".

## *this* Revisited

Consider:

```Js
var homework = {
    study() {
        console.log(`Please study${this.topic}`);
    }
};
var jsHomework=Object.create(homework);
jsHomework.topic="JS";
jsHomework.study();
// Please study JS

var mathHomework = Object.create(homework);
mathHomework.topic="Math";
mathHomework.study();
// Please study Math
```

The two objects *jsHomework* and *mathHomework* each prototype link to the single *homework* object, which has the *study()* function. *jsHomework* and *mathHomework* are each given their own *topic* property 

*jsHomework.study()* delegates to *homework.study()*, but its *this(this.topic)* for that execution resolves to *jsHomework* because of how the function is called, so *this.topic* is "JS". Similarly for *mathHomework.study()* delegating to *homework.study()* but still resolving *this* to *mathHomework*, and thus *this.topic* as "Math".
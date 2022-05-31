##  Values vs. References
InChapter 2, we introduced the two main types of values: primitives and objects

In many languages,the developer can choose between assigning/passing a value as the value itself, or as a reference to the value. In JS, however, this decision is entirely determined by the kind of value.

If you assign/pass a value itself, the value is copied. For example:
```Js
var myName = "Kyle";
var yourName = myName;
```

Here, the *yourName* variable has a separate copy of the *"Kyle"* string from the value that’s stored in *myName*. That’sbecause the value is a primitive, and primitive values arealways assigned/passed as **value copies**.

Here’s how you can prove there’s two separate values involved:
```Js
var myName = "Kyle";
var yourName = myName;
myName = "Frank";
console.log(myName);
// Frank
console.log(yourName);
// Kyle
```
Bycontrast,references are the idea that two or more variables are pointing at the same value, such that modifying this shared value would be reflected by an access via any of those references.In JS, only object values (arrays,objects,functions,etc.) are treated as references.
```Js
var myAddress={
    street:"123 JS Blvd",
    city:"Austin",
    state:"TX"
};
var yourAddress = myAddress;
// I've got to move to a new house!
myAddress.street="456 TS Ave";
console.log(yourAddress.street);
// 456 TS Ave
```
Because the value assigned to *myAddress* is an object, it’sheld/assigned by reference, and thus the assignment to the *yourAddress* variable is a copy of the reference, not the object value itself. That’s why the updated value assigned to the *myAddress.street* is reflected when we access your *Address.street*.*myAddress* and *yourAddress* have copies of the reference to the single shared object, so an update to one is an update to both.
## So Many Function Forms
```Js
var awesomeFunction = function(coolThings) {
    // ..
    return amazingStuff;
};
```
The function expression here is referred to as *ananonymous function expression*, since it has no name identifier between the *function* keyword and the (..) parameter list. This point confuses many JS developers because as of ES6, JS performs a “name inference” on an anonymous function:
```Js
awesomeFunction.name;
// "awesomeFunction"
```
The name property of a function will reveal either its directly given name (in the case of a declaration) or its inferred name in the case of an anonymous function expression. That value is generally used by developer tools when inspecting a function value or when reporting an error stack trace.

Even if a name is inferred,**it’s still an anonymous function.**
```Js
// let awesomeFunction = ..
// const awesomeFunction = ..
var awesomeFunction = function someName(coolThings) {
    // ..
    return amazingStuff;
};
awesomeFunction.name;
// "someName"
```
This function expression is a *named function expression*,since the identifier *someName* is directly associated with the function expression at compile time; the association with the identifier *awesomeFunction* still doesn’t happen until runtime at the time of that statement. 

Here are some more declaration forms:
```Js
// generator function declaration
function *two() { .. }
// async function declaration
async function three() { .. }
// async generator function declaration
async function *four() { .. }
// named function export declaration (ES6 modules)
export function five() { .. }

// IIFE
(function(){ .. })();
(functionnamedIIFE(){ .. })();
// asynchronous IIFE
(asyncfunction(){ .. })();
(asyncfunctionnamedAIIFE(){ .. })();
// arrow function expressions
var f;
f=() =>42;
f=x => x*2;
f=(x) => x*2;
f=(x,y) => x*y;
f=x => ({ x:x*2});
f=x => {return x*2; };
f=async x => {
    var y = await doSomethingAsync(x);
    return y*2;
};
someOperation( x => x*2);
// ..
```

Keep in mind that arrow function expressions are **syntactically anonymous**, meaning the syntax doesn’t provide away to provide a direct name identifier for the function. The function expression may get an inferred name, but only if it’s one of the assignment forms,not in the(more common!) form of being passed as a function call argument (as in the last lineof the snippet).

Functions can also be specified in class definitions and object literal definitions. They’re typically referred to as “methods”when in these forms, though in JS this term doesn’t have much observable difference over “function”:
```Js
classSomethingKindaGreat {
    // class methods
    coolMethod() { .. } // no commas!
    boringMethod() { .. }
}
var EntirelyDifferent = {
    // object methods
    coolMethod() { .. },// commas!
    boringMethod() { .. },

    // (anonymous) function expression property
    oldSchool: function() { .. }
};
```

## Coercive Conditional Comparison
*if* and *? :*-ternary statements, as well as the test clauses in *while* and *for* loops, all perform an implicit value compari-son. But what sort? Is it “strict” or “coercive”? Both, actually.

Consider:

```Js
var x = "hello";
if (x) {
    // will run!
}
if (x == true) {
    // won't run :(
}
```
Oops. So what is theifstatement actually doing? This is themore accurate mental model:
```Js
var x = "hello";
if (Boolean(x) == true) {
    // will run
}
// which is the same as:
if (Boolean(x) === true) {
    // will run
}
```
Since the *Boolean(..)* function always returns a value of type boolean,the *==* vs *===* in this snippet is irrelevant;they’ll both do the same thing. But the important part is to see that before the comparison,a coercion occurs,from whatever type *x* currently is, to boolean.

You just can’t get away from coercions in JS comparisons.Buckle down and learn them.
## Prototypal “Classes”
Another way of wiring up such prototype linkages served as the (honestly, ugly) predecessor to the elegance of the ES6 *class* system (seeChapter 2, “Classes”), and is referred to as prototypal classes.

Let’s first recall the *Object.create(..)* style of coding:
```Js
var Classroom = {
    welcome() {
        console.log("Welcome, students!");
    }
};
var mathClass = Object.create(Classroom);
mathClass.welcome();
// Welcome, students!
```
Here, a *mathClass* object is linked via its prototype to a *Classroom* object. Through this linkage, the function call *mathClass.welcome()* is delegated to the method defined on *Classroom.*

The prototypal class pattern would have labeled this delegation behavior “inheritance,” and alternatively have defined it(with the same behavior) as:
```Js
function Classroom() {
    // ..
}
Classroom.prototype.welcome = function hello() {
    console.log("Welcome, students!");
};
var mathClass = new Classroom();
mathClass.welcome();
// Welcome, students!
```
All functions by default reference an empty object at a property named *prototype*.Despite the confusing naming,this is **not** the function’s *prototype*(where the function is prototype linked to), but rather the prototype object to *link to* when other objects are created by calling the function with *new*.

We add a welcome property on that empty object (called *Classroom.prototype*), pointing at the *hello()* function.

Then *new Classroom()* creates a new object (assigned to *mathClass*), and prototype links it to the existing *Classroom.prototype* object.

Though *mathClass* does not have a *welcome()* property/-function, it successfully delegates to the function *Classroom.prototype.welcome().*

This “prototypal class” pattern is now strongly discouraged,in favor of using ES6’s *class* mechanism:
```Js
class Classroom {
    constructor() {
        // ..
    }
    welcome() {
        console.log("Welcome, students!");
    }
}
var mathClass = new Classroom();
mathClass.welcome();
// Welcome, students!
```
Under the covers, the same prototype linkage is wired up, but this *class* syntax fits the class-oriented design pattern much more cleanly than “prototypal classes”.
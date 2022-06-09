## When Can I Use a Variable?
```Js
greeting();
// Hello!

function greeting() { 
    console.log("Hello!");
}
```
This code works fine. 
Recall Chapter 1 points out that all identifiers are registered to their respective scopes during compile time. Moreover, every identifier is created at the beginning of the scope it belongs to, **every time that scope is entered.**

The term most commonly used for a variable being visible from the beginning of its enclosing scope, even though its declaration may appear further down in the scope, is called **hoisting.**

In other words, how does the variable *greeting* have any value (the function reference) assigned to it, from the moment the scope starts running? The answer is a special characteris- tic of formal *function* declarations, called *function hoisting*. When a *function* declaration’s name identifier is registered at the top of its scope, it’s additionally auto-initialized to that function’s reference. That’s why the function can be called throughout the entire scope!

One key detail is that both *function hoisting* and *var-*flavored *variable hoisting* attach their name identifiers to the nearest enclosing **function scope** (or, if none, the global scope), not a block scope.

> Declarations with *let* and *const* still hoist (see the TDZ discussion later in this chapter). But these two declaration forms attach to their enclosing block rather than just an enclosing function as with *var* and *function* declarations. 

## Hoisting: Declaration vs. Expression
Function hoisting only applies to formal function declarations, not to *function* expression assignments.
```Js
greeting();
// TypeError

var greeting = function greeting() { 
    console.log("Hello!");
};
```
Line 1 (*greeting();*) throws an error. But the kind of error
thrown is very important to notice. A *TypeError* means we’re trying to do something with a value that is not allowed.

Pay close attention to the distinction here. A *function* declaration is hoisted **and initialized to its function value** (again, called *function hoisting*). A *var* variable is also hoisted, and then auto-initialized to *undefined*. Any subsequent *function* expression assignments to that variable don’t happen until that assignment is processed during runtime execution.

In both cases, the name of the identifier is hoisted. But the function reference association isn’t handled at initialization time (beginning of the scope) unless the identifier was created in a formal *function* declaration.

## Variable Hoisting
```Js
greeting = "Hello!";
console.log(greeting);
// Hello!

var greeting = "Howdy!";
```
There’s two necessary parts to the explanation:
- the identifier is hoisted,
- **and** it’s automatically initialized to the value *undefined*
 from the top of the scope.
 ## Hoisting: Yet Another Metaphor
 Rather than hoisting being a concrete execution step the JS engine performs, it’s more useful to think of hoisting as a visualization of various actions JS takes in setting up the program **before execution.**

 The typical assertion of what hoisting means: *lifting*—like lifting a heavy weight upward—any identifiers all the way to the top of a scope. The explanation often asserted is that the JS engine will actually *rewrite* that program before execution, so that it looks more like this:

 ```Js
var greeting; // hoisted declaration 
greeting = "Hello!"; // the original line 1 
console.log(greeting); // Hello!
greeting = "Howdy!"; // `var` is gone!
```
 The hoisting (metaphor) proposes that JS pre-processes the original program and re-arranges it a bit, so that all the decla- rations have been moved to the top of their respective scopes, before execution. Moreover, the hoisting metaphor asserts that *function* declarations are, in their entirety, hoisted to the top of each scope. Consider:
 ```Js
studentName = "Suzy";
greeting();
// Hello Suzy!

function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;
```
The “rule” of the hoisting metaphor is that function declara- tions are hoisted first, then variables are hoisted immediately after all the functions. Thus, the hoisting story suggests that
program is *re-arranged* by the JS engine to look like this:

```Js
function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;

studentName = "Suzy";
greeting();
// Hello Suzy!
```
I assert that hoisting *should* be used to refer to the **compile- time operation** of generating runtime instructions for the automatic registration of a variable at the beginning of its scope, each time that scope is entered.
## Re-declaration?
```Js
var studentName = "Frank"; 
console.log(studentName); 
// Frank

var studentName; 
console.log(studentName); // ???
```
What do you expect to be printed for that second message? Many believe the second var studentName has re-declared the variable (and thus “reset” it), so they expect undefined to be printed.

If you consider this program from the perspective of the hoisting metaphor, the code would be re-arranged like this for execution purposes:

```Js
var studentName;
var studentName; // clearly a pointless no-op!

studentName = "Frank";
console.log(studentName);
// Frank

console.log(studentName);
// Frank
```
It’s also important to point out that *var studentName;* doesn’t mean *var studentName = undefined;*, as most assume. Let’s prove they’re different by considering this variation of the program:

```Js
var studentName = "Frank"; 
console.log(studentName); // Frank

var studentName;
console.log(studentName); // Frank <--- still!

// let's add the initialization explicitly
var studentName = undefined; 
console.log(studentName); // undefined <--- see!?
```
A repeated var declaration of the same identifier name in a scope is effectively a do-nothing operation. Here’s another illustration, this time across a function of the same name:
```Js
var greeting;

function greeting() { 
    console.log("Hello!");
}

// basically, a no-op
var greeting;
typeof greeting; // "function"
var greeting = "Hello!";
typeof greeting; // "string"
```
The first *greeting* declaration registers the identifier to the scope, and because it’s a *var* the auto-initialization will be *undefined*. The *function* declaration doesn’t need to re- register the identifier, but because of *function hoisting* it overrides the auto-initialization to use the function reference. The second *var greeting* by itself doesn’t do anything since *greeting* is already an identifier and *function hoisting* already took precedence for the auto-initialization.

What about repeating a declaration within a scope using *let* or *const*?
```Js
let studentName = "Frank"; 
console.log(studentName); 
let studentName = "Suzy";
```
This program will not execute, but instead immediately throw a *SyntaxError*. Depending on your JS environment, the error message will indicate something like: “studentName has already been declared.” In other words, this is a case where attempted “re-declaration” is explicitly not allowed!

It’s not just that two declarations involving *let* will throw this error. If either declaration uses *let*, the other can be either *let* or *var*, and the error will still occur, as illustrated with these two variations:
```Js
var studentName = "Frank"; 
let studentName = "Suzy"; 
```
and:
```Js
let studentName = "Frank"; 
var studentName = "Suzy";
```
In both cases, a *SyntaxError* is thrown on the second declaration. In other words, the only way to “re-declare” a variable is to use var for all (two or more) of its declarations.

When *Compiler* asks *Scope Manager* about a declaration, if that identifier has already been declared, and if either/both declarations were made with *let*, an error is thrown. The intended signal to the developer is “Stop relying on sloppy re-declaration!”
## Constants?
The *const* keyword is more constrained than *let*. Like *let*, *const* cannot be repeated with the same identifier in the same scope.

The *const* keyword requires a variable to be initialized, so omitting an assignment from the declaration results in a *SyntaxError*:
```Js
const empty; // SyntaxError
```
*const* declarations create variables that cannot be re-assigned:
```Js
const studentName = "Frank"; 
console.log(studentName); 
// Frank
studentName = "Suzy"; // TypeError
```
Any *const* “re-declaration” would also necessarily be a *const* re-assignment, which can’t be allowed!
```Js
const studentName = "Frank";

// obviously this must be an error
const studentName = "Suzy";
```
## Loops
```Js
var keepGoing = true; 
while (keepGoing) {
    let value = Math.random(); 
    if (value > 0.5) {
        keepGoing = false; 
    }
}
```
Is *value* being “re-declared” repeatedly in this program? Will
we get errors thrown? No.

Each loop iteration is its own new scope instance, and within each scope instance, *value* is only being declared once. So there’s no attempted “re-declaration,” and thus no error. Before we consider other loop forms, what if the *value* declaration in the previous snippet were changed to a *var*?
```Js
var keepGoing = true; 
while (keepGoing) {
    var value = Math.random(); 
    if (value > 0.5) {
        keepGoing = false; 
    }
}
```
Is *value* being “re-declared” here, especially since we know
*var* allows it? No. Because *var* is not treated as a block-scoping declaration (see Chapter 6), it attaches itself to the global scope. So there’s just one *value* variable, in the same scope as *keepGoing* (global scope, in this case). No “redeclaration” here, either!

What about “re-declaration” with other loop forms, like *for* loops?
```Js
for (let i = 0; i < 3; i++) {
    let value = i * 10; 
    console.log(`${ i }: ${ value }`);
}
// 0: 0
// 1: 10
// 2: 20
```
It should be clear that there’s only one value declared per scope instance. But what about i? Is it being “re-declared”?

To answer that, consider what scope *i* is in. It might seem like it would be in the outer (in this case, global) scope, but it’s not. It’s in the scope of *for*-loop body, just like *value* is. 

Now it should be clear: the i and value variables are both de- clared exactly once **per scope instance**. No “re-declaration” here.
```Js
for (let index in students) { 
    // this is fine
}
for (let student of students) { 
    // so is this
}
```
Same thing with *for..in* and *for..of* loops: the declared variable is treated as *inside* the loop body, and thus is handled per iteration (aka, per scope instance). No “re-declaration.”

Consider:
```Js
var keepGoing = true; 
while (keepGoing) {
    // ooo, a shiny constant!
    const value = Math.random(); 
    if (value > 0.5) {
        keepGoing = false; 
    }
}
```
Just like the *let* variant of this program we saw earlier, *const* is being run exactly once within each loop iteration, so it’s safe from “re-declaration” troubles. 

*for..in* and *for..of* are fine to use with const:
```Js
for (const index in students) { 
    // this is fine
}
for (const student of students) { 
    // this is also fine
}
```
But not the general *for*-loop:
```Js
for (const i = 0; i < 3; i++) {
    // oops, this is going to fail with
    // a Type Error after the first iteration
}
```
What’s wrong here? We could use *let* just fine in this construct, and we asserted that it creates a new *i* for each loop iteration scope, so it doesn’t even seem to be a “re-declaration.”

Let’s mentally “expand” that loop like we did earlier:
```Js
{
    // a fictional variable for illustration
    const $$i = 0;
    for ( ; $$i < 3; $$i++) {
        // here's our actual loop `i`! const i = $$i;
        // ..
    }
}
```
Do you spot the problem? Our *i* is indeed just created once inside the loop. That’s not the problem. The problem is the conceptual *$$i* that must be incremented each time with the *$$i++* expression. That’s **re-assignment** (not “re-declaration”), which isn’t allowed for constants.

The straightforward answer is: *const* can’t be used with the classic *for-loop* form because of the required re-assignment.
## Uninitialized Variables (aka, TDZ)
Consider:
```Js
console.log(studentName);
// ReferenceError

let studentName = "Suzy";
```
The result of this program is that a *ReferenceError* is thrown on the first line. Depending on your JS environment, the error message may say something like: “Cannot access studentName before initialization.”

That error message is quite indicative of what’s wrong: *studentName* exists on line 1, but it’s not been initialized, so it cannot be used yet. Let’s try this:
```Js
studentName = "Suzy";   // let's try to initialize it!
// ReferenceError
console.log(studentName); let studentName;
```
Oops. We still get the *ReferenceError*, but now on the first line where we’re trying to assign to (aka, initialize!) this so-called “uninitialized” variable *studentName*. What’s the deal!?

The real question is, how do we initialize an uninitialized variable? For *let/const*, the **only way** to do so is with an assignment attached to a declaration statement. An assign- ment by itself is insufficient! Consider:
```Js
let studentName = "Suzy"; 
console.log(studentName); // Suzy
```
Remember that we’ve asserted a few times so far that *Compiler* ends up removing any *var/let/const* declarators, replacing them with the instructions at the top of each scope to register the appropriate identifiers.

So if we analyze what’s going on here, we see that an additional nuance is that *Compiler* is also adding an instruction in the middle of the program, at the point where the variable *studentName* was declared, to handle that declaration’s autoinitialization. We cannot use the variable at any point prior to that initialization occuring. The same goes for *const* as it does for *let*.

The term coined by TC39 to refer to this *period of time* from the entering of a scope to where the auto-initialization of the variable occurs is: Temporal Dead Zone (TDZ).

The TDZ is the time window where a variable exists but is still uninitialized, and therefore cannot be accessed in any way. Only the execution of the instructions left by *Compiler* at the point of the original declaration can do that initialization. After that moment, the TDZ is done, and the variable is free to be used for the rest of the scope.

A *var* also has technically has a TDZ, but it’s zero in length and thus unobservable to our programs! Only *let* and *const* have an observable TDZ.
```Js
askQuestion();
// ReferenceError

let studentName = "Suzy";
function askQuestion() {
    console.log(`${ studentName }, do you know?`);
}
```
Even though positionally the *console.log(..)* referencing studentName comes *after* the *let studentName* declaration, timing wise the *askQuestion()* function is invoked *before* the let statement is encountered, while *studentName* is still in its TDZ! Hence the error.

There’s a common misconception that TDZ means *let* and *const* do not hoist. This is an inaccurate, or at least slightly misleading, claim. They definitely hoist.

We’ve already seen that *let* and *const* don’t auto-initialize at the top of the scope. But let’s prove that *let* and *const* do hoist (auto-register at the top of the scope), courtesy of our friend shadowing (see “Shadowing” in Chapter 3):
```Js
var studentName = "Kyle"; {
    console.log(studentName);
    // ???
    // ..
let studentName = "Suzy";
    console.log(studentName);
    // Suzy
}
```

The first *console.log(..)* throws a TDZ error, because in fact, the inner scope’s *studentName* **was** hoisted (auto-registered at the top of the scope). What **didn’t** happen (yet!) was the auto-initialization of that inner *studentName*; it’s still uninitialized at that moment, hence the TDZ violation!

So to summarize, TDZ errors occur because *let/const* dec- larations *do* hoist their declarations to the top of their scopes, but unlike *var*, they defer the auto-initialization of their variables until the moment in the code’s sequencing where the original declaration appeared. This window of time (hint: temporal), whatever its length, is the TDZ.

My advice: always put your *let* and *const* declarations at the top of any scope. Shrink the TDZ window to zero (or near zero) length, and then it’ll be moot.

## Finally Initialized
Hoisting is generally cited as an explicit mechanism of the JS engine, but it’s really more a metaphor to describe the various ways JS handles variable declarations during compilation. But even as a metaphor, hoisting offers useful structure for thinking about the life-cycle of a variable—when it’s created, when it’s available to use, when it goes away.

The TDZ (temporal dead zone) error is strange and frustrating when encountered. Fortunately, TDZ is relatively straightforward to avoid if you’re always careful to place *let/const* declarations at the top of any scope.

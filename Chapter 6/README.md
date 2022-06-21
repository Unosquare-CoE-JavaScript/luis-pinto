## Least Exposure
Software engineering articulates a fundamental discipline, typically applied to software security, called “The Principle of Least Privilege” (POLP). And a variation of this principle that applies to our current discussion is typically labeled as “Least Exposure” (POLE).

POLP expresses a defensive posture to software architecture: components of the system should be designed to function with least privilege, least access, least exposure. If each piece is connected with minimum-necessary capabilities, the over- all system is stronger from a security standpoint, because a compromise or failure of one piece has a minimized impact on the rest of the system.

If POLP focuses on system-level component design, the POLE *Exposure* variant focuses on a lower level; we’ll apply it to how scopes interact with each other.

When variables used by one part of the program are exposed to another part of the program, via scope, there are three main hazards that often arise:
- **Naming Collisions:** if you use a common and useful variable/function name in two different parts of the program, but the identifier comes from one shared scope (like the global scope), then name collision occurs, and it’s very likely that bugs will occur as one part uses the variable/function in a way the other part doesn’t expect.
- **Unexpected Behavior:** if you expose variables/functions whose usage is otherwise *private* to a piece of the program, it allows other developers to use them in ways you didn’t intend, which can violate expected behavior and cause bugs.
- **UnintendedDependency:** if you expose variables/functions unnecessarily, it invites other developers to use and depend on those otherwise *private* pieces. While that doesn’t break your program today, it creates a refactoring hazard in the future, because now you can- not as easily refactor that variable or function without potentially breaking other parts of the software that you don’t control.

Consider:
```Js
function diff(x,y) { if (x > y) {
let tmp = x; x = y;
y = tmp;
}
return y - x; }
diff(3,7);      // 4
diff(7,5);      // 2
```
In this simple example, it doesn’t seem to matter whether *tmp* is inside the *if* block or whether it belongs at the function level—it certainly shouldn’t be a global variable! However, following the POLE principle, *tmp* should be as hidden in scope as possible. So we block scope *tmp* (using *let*) to the *if* block.

## Hiding in Plain (Function) Scope
We’ve already seen the *let* and *const* keywords, which are block scoped declarators; we’ll come back to them in more detail shortly. But first, what about hiding *var* or *function* declarations in scopes? That can easily be done by wrapping a *function* scope around a declaration.

Let’s consider an example where *function* scoping can be useful.
```Js
var cache = {};
function factorial(x) { 
    if (x < 2) return 1; 
    if (!(x in cache)) {
        cache[x] = x * factorial(x - 1);
    }
    return cache[x]; 
}
factorial(6);
// 720
cache;
// {
// "2": 2,
// "3": 6,
// "4": 24,
// "5": 120,
//     "6": 720
// }
factorial(7);
// 5040
```
We’re storing all the computed factorials in cache so that across multiple calls to *factorial(..)*, the previous computations remain. But the *cache* variable is pretty obviously a *private* detail of how *factorial(..)* works, not something that should be exposed in an outer scope—especially not the global scope.
## Invoking Function Expressions Immediately
```Js
var factorial = (function hideTheCache() {      
    var cache = {};
    function factorial(x) { 
        if (x < 2) return 1; 
        if (!(x in cache)) {
            cache[x] = x * factorial(x - 1);
    }
    return cache[x]; }
    return factorial; 
})();
factorial(6);
// 720
factorial(7);
// 5040
```
Notice that we surrounded the entire *function* expression in a set of ( .. ), and then on the end, we added that second () parentheses set; that’s actually calling the *function* expression we just defined. 

So, in other words, we’re defining a *function* expression that’s then immediately invoked. This common pattern has a (very creative!) name: Immediately Invoked Function Expression (IIFE).

## Function Boundaries
For example, a *return* statement in some piece of code would change its meaning if an IIFE is wrapped around it,because now the *return* would refer to the IIFE’s function. Non-arrow function IIFEs also change the binding of a *this* keyword—more on that in the *Objects & Classes* book. And statements like *break* and *continue* won’t operate across an IIFE function boundary to control an outer loop or block.

So, if the code you need to wrap a scope around has *return, this, break, or continue* in it, an IIFE is probably not the best approach. In that case, you might look to create the scope with a block instead of a function.
## Scoping with Blocks
In general, any { .. } curly-brace pair which is a statement will act as a block, but **not necessarily** as a scope.

A block only becomes a scope if necessary, to contain its block-scoped declarations (i.e., *let* or *const*). Consider:
```Js
{
    // not necessarily a scope (yet)
    // ..
    // now we know the block needs to be a scope
    let thisIsNowAScope = true;
    for (let i = 0; i < 5; i++) {
        // this is also a scope, activated each 
        // iteration
        if (i % 2 == 0) {
        // this is just a block, not a scope 
        console.log(i);
        } 
    }
}
// 0 2 4
```
If you find yourself placing a *let* declaration in the middle of a scope, first think, “Oh, no! TDZ alert!” If this *let* declaration isn’t needed in the first half of that block, you should use an inner explicit block scope to further narrow its exposure!
## *var* and *let*
*var* is visually distinct from *let* and therefore signals clearly, “this variable is function-scoped.” Using *let* in the top-level scope, especially if not in the first few lines of a function, and when all the other declarations in blocks use *let*, does not visually draw attention to the difference with the function- scoped declaration.
## Where To *let*?
 If you decide initially that a variable should be block-scoped, and later realize it needs to be elevated to be function-scoped, then that dictates a change not only in the location of that variable’s declaration, but also the declarator keyword used. The decision-making process really should proceed like that.

 If a declaration belongs in a block scope, use let. If it belongs in the function scope, use *var* (again, just my opinion).

 Following this perspective, you can find any *var* that’s inside a block of this sort and switch it to *let* to enforce the semantic signal already being sent. That’s proper usage of *let* in my opinion.
## What’s the Catch?

```Js
try { 
    doesntExist();
}
catch (err) {
    console.log(err);
    // ReferenceError: 'doesntExist' is not defined
    // ^^^^ message printed from the caught exception
    let onlyHere = true;
    var outerVariable = true; 
}
console.log(outerVariable);     // true
console.log(err);
// ReferenceError: 'err' is not defined
// ^^^^ this is another thrown (uncaught) exception
```
The *err* variable declared by the *catch* clause is block-scoped to that block. This *catch* clause block can hold other block- scoped declarations via *let*. But a *var* declaration inside this block still attaches to the outer function/global scope.
## Function Declarations in Blocks (FiB)

We typically think of *function* declarations like they’re the equivalent of a *var* declaration. So are they function-scoped like *var* is?

No and yes. I know... that’s confusing. Let’s dig in:
```Js
if (false) { 
    function ask() {
        console.log("Does this run?");
    }
} 
ask();
```
What do you expect for this program to do? Three reasonable outcomes:

1. The *ask()* call might fail with a *ReferenceError* exception, because the ask identifier is block-scoped to the *if* block scope and thus isn’t available in the outer/global scope.
2. The *ask()* call might fail with a *TypeError* exception, because the *ask* identifier exists, but it’s *undefined* (since the *if* statement doesn’t run) and thus not a callable function.
3. The *ask()* call might run correctly, printing out the “Does it run?” message.

The JS specification says that *function* declarations inside of blocks are block-scoped, so the answer should be (1). However, most browser-based JS engines (including v8, which comes from Chrome but is also used in Node) will behave as (2), meaning the identifier is scoped outside the *if* block but the function value is not automatically initialized, so it remains *undefined*.
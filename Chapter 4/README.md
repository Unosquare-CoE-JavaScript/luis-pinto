This final chapter divides the organization of the JS language into three main pillars
## Pillar 1: Scope and Closure
The organization of variables into units of scope (functions,blocks) is one of the most foundational characteristics of any language

Scopes are like buckets, and variables are like marbles youput into those buckets. The scope model of a language is like the rules that help you determine which color marbles go in which matching-color buckets.

Scopes nest inside each other,and for any given expression or statement, only variables at that level of scope nesting, or in higher/outer scopes, are accessible; variables from lower/inner scopes are hidden and inaccessible.

This is how scopes behave in most languages, which is called lexical scope. The scope unit boundaries, and how variables are organized in them, is determined at the time the program is parsed (compiled)

JS is lexically scoped, though many claim it isn’t, because of two particular characteristics of its model that are not present in other lexically scoped languages.

The first is commonly called *hoisting:* when all variables declared anywhere in a scope are treated as if they’re declared at the beginning of the scope.

The other is that *var-*declared variables are function scoped, even if they appear inside ablock.

*let/const* declarations have a peculiar error behavior called the “Temporal Dead Zone” (TDZ) which results in observable but unusable variables. Though TDZ can be strange to encounter, it’s *also* not an invalidation of lexical scoping. All of these are just unique parts of the language that should be learned and understood by all JS developers.

Closure is a natural result of lexical scope when the language has functions as first-class values,as JS does.When a function makes reference to variables from an outer scope, and that function is passed around as a value and executed in other scopes, it maintains access to its original scope variables; this is closure.

## Pillar 2: Prototypes
JS is one of very few languages where you have the option to create objects directly and explicitly, without first defining their structure in a class.

For many years, people implemented the class design pattern on top of prototypes—so-called “prototypal inheritance” and then with the advent of ES6’s *class* keyword, the language doubled-down on its inclination toward OO/class-style programming.

But I think that focus has obscured the beauty and power of the prototype system: the ability for two objects to simply connect with each other and cooperate dynamically (during function/method execution) through sharing a *this* context.

Classes are just one pattern you can build on top of such power. But another approach, in a very different direction, is to simply embrace objects as objects,forget classes altogether,and let objects cooperate through the prototype chain. This is called *behavior delegation*. I think delegation is more powerful than class inheritance, as a means for organizing behavior and data in our programs.

I encourage you to spend plenty of time deep in Book 3,*Objects & Classes*, to see how object delegation holds far more potential than we’ve perhaps realized.This isn’t an anti-*class* message,but it is intentionally a “classes aren’t the only way to use objects” message that I want more JS developers to consider.

Object delegation is, I would argue, far more *with the grain* of JS, than classes

## Pillar 3: Types and Coercion
The vast majority of developers have strong misconceptions about how *types* work in programming languages, and especially how they work in JS. A tidal wave of interest in the broader JS community has begun to shift to “static typing”approaches,using type-aware tooling like TypeScript or Flow.

Arguably, this pillar is more important than the other two,in the sense that no JS program will do anything useful if it doesn’t properly leverage JS’s value types, as well as theconversion (coercion) of values between types.

## With the Grain
First, consider the *grain*(as in, wood) of how most people ap-proach and use JS. 

In other words, don’t be afraid to go against the grain, as I have done with these books and all my teachings. Nobody can tell you how you will best make use of JS; that’s for you to decide. I’m merely trying to empower you in coming to your own conclusions, no matter what they are.

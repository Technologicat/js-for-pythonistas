*To read this README offline in a sanely formatted way, [grip](https://github.com/joeyespo/grip) it, or [Pandoc](https://pandoc.org/) it into your preferred file format.*

## ECMAScript for Python developers in 30 minutes

ECMAScript, formerly known as JavaScript, is still the only language supported by web browsers, so for the frontend of a web app it is a requirement. Things are slowly changing ([Brython](https://brython.info/), [WebAssembly](https://webassembly.org/)), but as of early 2020, ES remains the dominant language for the frontend, as well as the implementation language under the hood of the alternatives.

Here's a quick overview for those coming from Python, to hit the ground running. Some experience with the Lisp family helps, but is not a requirement.

### Summary

ECMAScript is no longer the [Turing tarpit](https://en.wikipedia.org/wiki/Turing_tarpit) it used to be. Modern ECMAScript (ES6 and beyond) is essentially like Python, but with a lot more semicolons and curly braces, and with (more than?) half of the standard library missing. As a pythonista, you'll *almost* feel right at home.

If you're hardcore, and have [copious free time](http://catb.org/jargon/html/C/copious-free-time.html), read [the standard](https://www.ecma-international.org/ecma-262/). Also all of [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference), and for good measure, [w3schools.com](https://www.w3schools.com/jsref/default.asp).


### Getting started

Just like Python, ES is a duck-typed, dynamic language. It's an interpreted language in the same modern sense as Python: ES compiles to bytecode, which then runs on the ES virtual machine. There are several implementations (*engines*), for example [SpiderMonkey](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Internals/Bytecodes) in Firefox, and [V8](https://medium.com/dailyjs/understanding-v8s-bytecode-317d46c94775) in Chrome and in Node.js.

Truthiness semantics are similar to Python's, with similar short-circuiting logic. The logic operators are spelled as in C, `&&` (**and**), `||` (**or**), `!` (**not**). The constants `true` and `false` are spelled in lowercase. The constant `null` is ES's equivalent of Python's `None`. There is also [the constant `undefined`](https://codeburst.io/javascript-null-vs-undefined-20f955215a2), which ES itself uses to indicate e.g. that a variable has been declared, but not initialized. As an argument to a function call, `undefined` means *use the default value for the parameter*.

[Full list of ES operators on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this).

Parentheses around the check in `if` are mandatory. The spelling for chaining `if`s is `else if`, and the bare `else` is available as usual. Like in C, curly braces around the body (a.k.a. suite) are optional if there is only one statement in the body. **Unlike in Python**, for multiple statements the braces are required; **in ES indentation is for humans only**.

Basic containers are `Array` (like Python's `list`), `Set` (like Python's `set`), and... the object. There's no separate `dict` type, `Object` itself acts like one. Attributes (ES calls them *properties*) can be accessed with `obj.x` as well as `obj['x']`; the latter is handy if the name is stored in a variable. There's no generic `__getattr__` override; unlike Python, ES doesn't go to *superhuman lengths to expose its moving parts* ([Evelyn Woods](https://eev.ee/blog/2017/11/28/object-models/)). You can define Python-style properties, though, triggering computation on access to a specific attribute - this feature is known as *accessor properties*, a.k.a. *getters and setters*.

Strings are quoted using `"..."` or `'...'`. Just like in Python, they do the same thing, and avoid the need to escape the other type of quote inside the string. **Unlike in Python, there is no `"""..."""`**.

The equivalent of Python's [f-strings](https://www.python.org/dev/peps/pep-0498/) are [*template literals*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals). They are quoted using backticks `` `...` ``, and `${...}` inside a template literal interpolates an expression. For example, ``const s = `Hello, ${name}!`;`` interpolates the value of the variable `name` into the string. (The exclamation mark is just part of the string; only the `${...}` is treated specially.)

ES6 introduced a pythonic module system. Beware though, **if you *don't* use modules, the global top-level scope is shared**, unlike in Python. If you want to try modules, read up on `import` and `export`, and in the HTML `<script>` tag, use `type="module"` instead of `type="text/javascript"`. Some sources recommend, as a convention, the use of the file extension `.mjs` instead of the traditional `.js` for module-based code.

The classical, pre-ES6 solution that you'll still often see in the wild, is *the module pattern*: to emulate modules using an *IIFE* (*immediately invoked function expression*). The IIFE approach takes advantage of [the closure property](https://en.wikipedia.org/wiki/Closure_(computer_programming)) to hide internal details from the shared top-level scope. Such an old-style "module" typically contains code at the end of the function to explicitly inject an object into the top-level scope. This object is, by convention, named after the "module", and contains its exports.

Unlike Python, **ES has a proper lambda** (for some reason, known as [`function`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions); no nod to [history](http://matt.might.net/articles/compiling-up-to-lambda-calculus/)?), that can contain several statements **and use all features of the language**. Even in Python, the single-expression limitation of `lambda` is really no limitation, see [`unpythonic`](https://github.com/Technologicat/unpythonic) for two different ways to place multiple expressions there. Instead, what bites us in Python is that a Python `lambda` only accepts *expressions*, and many language features of Python are only available as *statements*. **ES does not go the extra mile to shoot itself in the foot like Python does**. ES supports both *function statements* (for defining a named function) and *function expressions* (for use as a λ). Both use the same keyword, `function`, and *whatever you can put in the body of a function statement, you can put in the body of a function expression*.

The practical advantage is that a proper λ is often useful for achieving a top-down presentation order, especially when callbacks are involved. You can first say what your goal is, and then right there in the argument list of the higher-order function you're calling, write the callback λ that handles the details - instead of having to define a named function up front, before the overly rigid shape of the language even allows you to state what you're writing about. If λ being anonymous is a problem for human-readable tracebacks, then the language should allow naming function expressions, not cripple the λ construct, dammit.

**ES gets this right:** [**you can name a function expression**](https://raganwald.com/2014/10/24/fun-with-named-functions.html). See [Named function expressions demystified](https://kangax.github.io/nfe/). As a bonus, the name will be bound inside the function (but not outside it!), which is useful for recursion. Also, if you write a declaration of the form `const f = function (...) {...};`, ES infers ([like Racket](https://docs.racket-lang.org/guide/define.html)) that the intent is to define a function named `f`, so it'll set the name for tracebacks, even though the function expression itself is anonymous.

On the other hand, a very nice feature of Python, which ES simply **does not have**, is passing arguments by name. [You can approximately emulate that](https://stackoverflow.com/questions/50511398/javascript-es6-named-parameters-and-default-values) if you try hard enough, but that's not very idiomatic. So be **very, very careful** about the ordering of arguments in function calls. Dynamic typechecking and no named arguments is a silly and dangerous combination, but that's ES for you.

**DANGER: In ES, providing the wrong number of arguments to a function call *fails silently***. No helpful `TypeError`, unlike in Python! If too few arguments were given, any leftover parameters are simply set to `undefined`. If too many arguments are given, the rest are just dropped on the floor. So be careful - the language won't help you. (This is probably how haskellers feel about Python.)

As a pythonista, you'll definitely want [**strict mode**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) (`'use strict';`), which e.g. requires declaring variables (to explicitly scope them and to ensure there are no typos). **It's the default for modules, but not for classical scripts**.

Just like Python, **ES does not have [tail call optimization](https://en.wikipedia.org/wiki/Tail_call)**, so you have to be careful about the maximum recursion depth. If arbitrarily deep tail-recursion is important to what you're doing, you could always define your own trampoline, [like Clojure does](https://clojuredocs.org/clojure.core/trampoline). See [`unpythonic.tco`](https://github.com/Technologicat/unpythonic/blob/master/unpythonic/tco.py) for how to do that in Python. The same technique applies trivially to ES.

The terminology in ES is a bit different from Python. For example, what Python calls attributes, ES calls *properties*. What Python calls properties, ES calls [*accessor properties*](https://stackoverflow.com/a/56144792). What Python calls unpacking, ES calls *destructuring*. (Dictionary unpacking is *object destructuring*.) What Python calls `lambda` (in the practical sense of a short, anonymous snippet, usually to be fed into a [higher-order function](https://en.wikipedia.org/wiki/Higher-order_function) such as `map`, `filter` or `reduce`), ES calls [*arrow functions*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions).

Language versions of modern ES are numbered starting from ES6, a.k.a. ES2015, based on the year it was released. As the preface of the standard says, ES6 was a huge 15-year standardization effort. Since then, there has been one release of [the language standard](https://www.ecma-international.org/ecma-262/) per year, each time adding 1 to the version number. So e.g. ES7 is ES2016, and ES10 is ES2019. These annual releases often contain a small number of updates, compared to the major overhaul that was ES6.


### Scoping of variables

[Lexical](https://en.wikipedia.org/wiki/Scope_(computer_science)#Lexical_scoping_vs._dynamic_scoping), just like in Python.

A nice feature in ES, that Python doesn't have, is fine-grained control of the scope and constness of variables ([names](https://nedbatchelder.com/text/names.html), really, in the same sense as in Python):

- `var`: scoped to the nearest enclosing function scope, like Python's locals,
- `let`: scoped to the nearest enclosing curly braces, and
- `const`: like `let`, but with **the language-level guarantee** that a `const` variable cannot be re-assigned to point to something else.

Of these, `let` and `const` (declared at the appropriate level) are the most useful, as they can be **scoped to exactly where you want**, avoiding pollution of a function's top-level scope by temporaries local to e.g. an `if` branch.

Unfortunately, due to historical reasons, the design is slightly botched. The `var` kind **may shoot you in the foot**:

- At the global top level (when not inside any function), **`var` will happily pollute the shared global scope**. (Good luck not trampling over anyone else's definitions by accident.)
- A `var` variable always [gets initialized to `undefined`](https://wsvincent.com/javascript-temporal-dead-zone/). It will **not** throw a `ReferenceError` if you try to access it in the declared-but-not-yet-defined state. This may smite you much later.

In contrast, **`let` and `const` are safe**:

- At the global top level, `let` and `const` [**do not pollute**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let) the shared global scope. They remain local to the file in which they are declared.
- Accessing a `let` or `const` value, inside the scope where it's declared, but before the assignment that sets its value has been executed, will throw a helpful `ReferenceError` (just like Python's `UnboundLocalError` in the analogous situation).

Example:

```javascript
function f () {
    console.log(x);  // ReferenceError!
    let x = 42;
}
```

The name `x` will be in scope immediately when `f` is entered, but it is only usable after the `x = 42` has been executed - as expected, if you're used to Python. The `let x` part instructs the ES compiler to allocate the variable `x` in the lexical scope where the `let` appears, whereas the `x = 42` takes place at runtime, when execution hits that line. The analogous situation in Python is:

```python
def f():
    print(x)  # UnboundLocalError!
    x = 42
```

This is completely analogous, because in the ES example above, the `let` appears in the top-level scope of the function `f`. Keep in mind that in ES, we could also `let` inside e.g. an `if` branch, which has no direct analogue in Python, because in Python the smallest unit of scope is the function. (In Python, fine-grained scoping is not available; with the technical detail that a `list`/`set`/`dict` comprehension is shorthand for an immediately called function that does a specific task.)

When inside a function, sometimes a strategically placed `var` can be useful to "leak", on purpose, a variable to the nearest enclosing function scope. If you want to use the technique of reading the final value of a loop counter after a loop finishes, you'll need a `var`, if you want to declare the counter variable in the `for` loop header. Just be careful not to read the value of the counter before it is assigned. Alternatively, you can `let` the name outside the loop beforehand, at the function scope, and then just set its value (not declare) it in the `for` loop header. This is one more line of code, but free of pitfalls.


### Looping over a collection

A [`for...of` loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of) is like Python's `for...in`.

**CAUTION**: ES also has the syntax `for...in`, but that's [a different thing](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in).

Like in C, curly braces around the loop body are optional if there is only one statement in the body. **Unlike in Python**, for multiple statements the braces are required; **in ES indentation is for humans only**.

This example:

```javascript
inp = [1, 2, 3, 4, 5];
out = [];
for (let x of inp) {
    out.push(function () {return 2*x;});
}
for (let f of out) {
    console.log(f());
}
```

will print 2, 4, 6, 8, 10, as expected, because using `let` in a `for` loop will create a fresh `x` for each iteration. (It can even be `const x of inp` if you don't need to re-assign the name `x` inside the loop body. It's a different `x` each iteration, so nothing is getting re-assigned by the loop itself.) If you use `var x of inp`, then it behaves like in Python. Then there is only one `x`, which lives in the function scope, and you'll get 10, 10, 10, 10, 10. Similarly if you `let x` at the function scope, and then `for (x of inp)`.

For objects acting as dictionaries, the equivalent of `d.keys()` is [`Object.keys(d)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys), for `d.values()` it's [`Object.values(d)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/values), and for `d.items()`, [`Object.entries(d)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries). You could use `for...in`, instead of `for...of` with `Object.keys`, but there is a difference: `for...in` also walks the prototype chain, whereas `Object.keys` skips that and only gives the keys for the object's own properties.

ES has generators, and `for...of` loops iterate over them. To define a generator, use `function*` (note the star; this seems a nod to naming conventions [in the Lisp family](https://docs.racket-lang.org/reference/let.html)) instead of `function`; you can then use `yield` in the body.

There is no `list`/`set`/`dict` comprehension syntax. For lists, see [`Array.prototype.map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) (in plain English, that's the method `map` of `Array` objects) and [`Array.prototype.filter`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter). So for example to create a list of even squares, this Python:

```python
evsq = [x**2 for x in range(10) if (x**2 % 2) == 0]
```

becomes this ES:

```javascript
const evsq = range(10).map(x => x**2).filter(y => y % 2 === 0);
```

The code is presented in the same order as it executes.  You can chain the `map` and `filter` operations in whatever order, depending on what makes the most sense for your use case. The [exponentiation operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Exponentiation) was added in ES7 (ES2016). In ES6 and below, `Math.pow(base, exponent)` was used instead.

**CAUTION**: I'm glossing over the fact there's no `range` in ES. But you can use the one we provide further below.

If you need a dictionary comprehension, one possibility is to define an analogue of `map` for objects acting as dictionaries:

```javascript
function dictmap (fkey, fval, olddict) {
    let newdict = {};
    for (const [key, val] of Object.entries(olddict))
        newdict[fkey(key, val)] = fval(key, val);
    return newdict;
}

const d1 = {a: 1, b: 2};
const d2 = dictmap((k, v) => k + k,  // how to make new key
                   (k, v) => v**2,   // how to make new value
                   d1);
}
```

I have chosen to include both `k` and `v` as parameters to both `fkey` and `fval` to emulate how Python scopes them in its `dict` comprehension (both `k` and `v` are available anywhere in the body expression).

Converting the code to use the implicit `this` instead of an `olddict` parameter, you could even monkey-patch such a function onto `Object.prototype`... but that's probably a bad idea.


### Equality comparison

For [equality comparisons](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness), you'll most often want to use `===` (**three** equals signs) and `!==`. They are the *strict equality* and *strict inequality* operators, which skip some implicit type conversions.

For custom objects, all of ES's equality comparison operators behave like Python's `is`:

```javascript
foo = {a: 1};
bar = {a: 1};
console.log(foo === bar);          // false
console.log(foo == bar);           // false
console.log(Object.is(foo, bar));  // false
```

If you want to make a custom by-value equality comparison for your own class (like Python's `__eq__` dunder override), [just write a custom function](http://adripofjavascript.com/blog/drips/object-equality-in-javascript.html) and call that. No nice infix syntax for you.

Be aware that **builtin types come in two flavors**, and this language implementation detail is not papered over as meticulously as in Python. For example, in ES, string literals (*primitive* strings) and `String` objects are considered different. To test whether a value is *a* string, use `(typeof obj === "string") || (obj instanceof String)`. Neither test alone will do.

- [`typeof obj`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof) identifies the [primitive type](https://developer.mozilla.org/en-US/docs/Glossary/Primitive) of `obj`. For anything that is not a primitive, the result is `"object"`.
- [`obj instanceof Foo`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof) tests whether the object was produced by the constructor function `Foo`, or anything that inherits from `Foo`.

(Maybe this is a good time to point out that in ES, strings internally use an [UCS-2 encoding with surrogates](https://mathiasbynens.be/notes/javascript-encoding). More about [Unicode in ES](https://mathiasbynens.be/notes/javascript-unicode).)


### Object model

Much has been said on the internet about the ECMAScript object model being foreign and weird to many developers - based on *prototypes*, not classes - but actually [that's pretty much exactly how Python works, too](https://eev.ee/blog/2017/11/28/object-models/).

Especially, the delegation mechanism is essentially the same. Recall that in Python, if an attribute is not found directly on the object, it is next looked up on the class, which is also just an object (essentially a prototype!), and if still not found, then up the [MRO](https://en.wikipedia.org/wiki/C3_linearization). This is exactly what ES does, with the small difference that unlike Python, **ES only supports single inheritance**. ES looks up a property on the object itself first, and if not found, then walks up the prototype chain.

Consider that in Python, you want to have your methods defined on the class, not on the instance. Python's `class` syntax conveniently automatically does this when you define a method in the `class` body. Similarly, in ES, you want your methods defined on the prototype, not on the instance. Similarly, this is conveniently automatically done when you use ES's [`class` syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes), introduced in ES6. See [[1]](https://levelup.gitconnected.com/using-classes-in-javascript-e677d248bb6e), [[2]](https://dmitripavlutin.com/javascript-classes-complete-guide/) for an overview.

The pre-ES6, classical way to define objects in JavaScript is to represent the object by a constructor function that returns the new instance. This seems a nod to the Lisp family, using the [closure-object-duality](http://people.csail.mit.edu/gregs/ll1-discuss-archive-html/msg03277.html). In the wild, you'll see functions that assign methods to `this.prototype.xxx`, so that's what it means. But for new code, it's perhaps clearer to use the `class` syntactic sugar. **C++ and Java programmers beware; `class` is just sugar, the ES object model remains prototype-based regardless of which syntax is used.**

The **naming convention** is that just like in Python, a class name starts with a capital letter. Similarly, if the object is defined using the constructor function syntax, the name of the function (by convention) starts with a capital letter.

Sometimes *object literals* are useful. An object literal looks almost exactly like a Python dictionary:

```javascript
obj = {a: 1, b: 2};
```

If a key is a valid *identifier name*, [it is not necessary to quote it](https://mathiasbynens.be/notes/javascript-properties). An identifier name is either a valid identifier (i.e. anything that can be used as a variable name) or a reserved word of the language. See the link for more details.

This is easy to remember by recognizing that **the keys *are* property names**. To read the "`a`" property of `obj`, one would write `obj['a']`, **or equivalently**, `obj.a`. So the object literal syntax just mirrors this.


### Destructuring (unpacking)

Unlike Python, [where the feature was removed in 3.0](https://www.python.org/dev/peps/pep-3113/), ES supports destructuring **also in a function parameter list**, both in regular function definitions and in arrow functions.

Square brackets pull elements from the beginning of an array:

```javascript
function f ([a, b]) {
    console.log(a);
    console.log(b);
}
arr = [1, 2];
f(arr);

g = ([a, b]) => a;
console.log(g(arr));  // 1
```

Here both `f` and `g` take a single argument, which must be an array.

Curly braces pull properties by name from an object:

```javascript
function f ({a, b}) {
    console.log(a);
    console.log(b);
}
obj = {a: 17, b: 42};
f(obj);

g = ({a, b}) => a;
console.log(g(obj));  // 17
```

Here both `f` and `g` take a single argument, which must be an object.

Just like Python's unpacking, ES's destructuring can be used to perform a pythonic swap:

```javascript
let a = 1;
let b = 2;
[a, b] = [b, a];
console.log(a);  // 2
console.log(b);  // 1
```


### Asynchronous I/O, multithreading

Keep in mind that as a general rule, I/O in ES is async. This applies also to the loading of scripts, and of any assets. Node.js and libraries for it (e.g. [lockfile](https://github.com/npm/lockfile)) make some exceptions here, but generally it is better to write async code, because that eliminates the I/O wait time, allowing the program to do something else while waiting for data to arrive (often, over the network!).

In ES, [Web workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) provide genuine OS threads, and they can be used for **true parallelism** (unlike genuine OS threads in Python, where [the GIL limits performance](http://python-notes.curiousefficiency.org/en/latest/python3/multicore_python.html)). They are usually defined in a separate `.js` file, but if that feels overly bureaucratic in a small program, it's possible to embed the code into the same file, by generating an object URL (see the MDN link). Web workers are best only used for computationally heavy background tasks that must not hang the UI.

Most ES code is just fine single-threaded, with the help of async I/O. In ES, async comes in many flavors. Traditionally callbacks were used, leading to nested callback hell (with the accompanying runaway indentation) when several async operations needed to be chained. The next evolution were Promises. But nowadays you'll want the modern syntactic sugar: `async function` and `await`, like in Python 3.5+. Unlike in Python, there is always an event loop, so an async function can be called normally from anywhere.

In ES, async functions are actually built on Promises, so understanding those first helps. See [JS for impatient programmers](https://exploringjs.com/impatient-js/ch_promises.html). When called normally, an async function will return a [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise). So essentially `async function` is just (highly convenient) syntactic sugar for defining a function that returns a `Promise`.

An ES promise has `then` and `catch` methods, which can be used to schedule code to run upon success and upon error. (`Promise` is essentially a Maybe monad. An error causes `Promise` to skip any success callbacks until it finds the next error handler.)

When lexically inside an async function, just like in Python, you can `await` another async function. This takes care of arranging things so that the code after the `await` runs after the result is in. That code essentially implements the `then` method for the `Promise`. For error handling, you can surround the `await` in a `try`/`catch`; [the exception handler will then act as the `catch` method](https://javascript.info/async-await) of the `Promise`. Just like in Python, the `async`/`await` syntactic sugar makes the code look like regular synchronous code, while it actually runs asynchronously.

If you find yourself in legacy callback hell, you may be interested in [promisification](https://javascript.info/promisify) (i.e. [converting callbacks to promises](https://zellwk.com/blog/converting-callbacks-to-promises/)). At the backend, Node.js already has [`util.promisify`](https://nodejs.org/api/util.html#util_util_promisify_original), but using the above information you can easily code your own for the frontend.


### File access in the frontend

For loading static assets, the simplest way is to use an `<object>` tag in the HTML document. Then you can just grab the object from the DOM **after** the document has finished loading. You'll want to set [`window.onload`](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event), or do [
something similar](https://javascript.info/onload-ondomcontentloaded).

When this is not an option (e.g. when the filename must be computed at runtime), you can use [XMLHttpRequest (a.k.a. XHR)](https://stackoverflow.com/questions/40201137/i-need-to-read-a-text-file-from-a-javascript) to load assets. There is usually a **same-origin policy** in effect, unless you enable [Cross-origin resource sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) in your HTTP server that's serving the app. In practice this means that using XHR, you can load assets from the directory your `.js` file lives in, as well as any of its subdirectories.

**Even for local testing, you need an HTTP server**. Just opening the `.html` locally in a web browser fails if your app uses XHR. For example, the `contentDocument` property of an `<object>` asset might never become available, when the `.html` is viewed directly from a file, because then the origin is `null`, and hence without CORS, XHR is not allowed. [When the same files are served by an HTTP server](https://stackoverflow.com/questions/50080561/why-can-i-not-acces-with-contentdocument-to-a-svg-file-in-chrome-but-i-can-in-fi), the origin is set, and it works fine.

For a quick HTTP server for testing, `python3 -m http.server` does the trick. Run it in the directory that has your `index.html`, and then browse to `http://localhost:8000`.

**The frontend does not support writing files**. There are three solutions, depending on what you need:

- [Let the user download the data as a file](https://shinglyu.com/web/2019/02/09/js_download_as_file.html), which they can then save locally.
  - This is convenient for exporting data, when there is no need to load that data back later, such as saving copies of plotted figures.

- Cookies, for storing small amounts of data on the client side. Best for unimportant temporary data.
  - Note any applicable privacy laws, and that cookies typically expire after a given deadline.
  - If your app is later moved to a different domain, typically it can no longer access the cookies from the old domain.
  - To allow the user to transfer such data manually, you could zip and base64 encode it, then allow the user to enter such a base64 encoded string to load the data back in.

- A custom server-side app, which can accept e.g. a `multipart/form-data` HTTP POST request, and save the data into a file at the backend.
  - See [Resumable.js](https://github.com/23/resumable.js) for a frontend library that can send data in that format. For a matching backend for Node.js, see [resumable-node](https://github.com/mrawdon/resumable-node).


### Error handling

Proper error handling mostly isn't a thing in the ES culture, but the language does support `try {...} catch (e) {...} finally {...}`. The parameter to `catch` is mandatory, until very recent versions of the language.

The curly braces are always mandatory, unlike in some other constructs such as `for` and `if`.

Catching a specific exception type isn't supported by the standard, but you can catch, test (type or duck), and re-throw when no match:

```javascript
try {
    some_failing_operation();
}
catch (e) {
    if (e instanceof MyError) {
        console.log("yay");
    }
    else {
        throw(e);
    }
}
```

There are very few [builtin exception types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error), but much of the time, even in Python, `TypeError` (ES has that too) and `ValueError` (ES: `RangeError`) are the most useful. If you want to make your own exception type to be able to catch that specifically:

```javascript
class MyError extends Error {
    constructor (...args) { super(...args); }
}
```

Unfortunately that's a bit more boilerplate than Python's `class MyError(Exception): pass`, but that's ES for you. The `super` call is really necessary even in simple cases like this.

Here the `...` is literal; it's the splat operator, like `*` in Python. In a parameter list, `...` defines a *rest parameter*, which must be the last one in the parameter list. It collects all remaining arguments into an array. In a function call, `...` unpacks an array into individual arguments. So, as a pythonista, you can mostly think of ES's `...` as the same as Python's unpacking `*`.


### Pitfalls

ES as a language has many pitfalls. (That's not to say Python doesn't have its share, such as mutable default arguments; the loop variable scoping thing that affects closures created in a loop; and the various [import traps](http://python-notes.curiousefficiency.org/en/latest/python_concepts/import_traps.html) for the unwary).

Examples:

- Syntax for function expressions vs. function statements. If the context allows a statement, that takes precedence.
  - Sometimes this means you must notice the parentheses around the construct vs. the lack thereof, or get smited.

- Implicit semicolons: [*automatic semicolon insertion (ASI)*](https://stackoverflow.com/questions/2846283/what-are-the-rules-for-javascripts-automatic-semicolon-insertion-asi). It may do what you mean, except sometimes not.
  - **Unlike in Python**, in ES newlines and indentation don't matter (except to humans); semicolons and curly braces do.
  - ASI is an attempt to paper over this, but the ES language wasn't originally designed to be written without semicolons, so the rules may seem arbitrary and be difficult to remember.

- The implicit `this` parameter, [whose value depends](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) on how the function was invoked.
  - In the wild, you may sometimes see the pattern `var that = this;`. There's nothing special about `that`; the intent is to store the `this` value from that scope for use in inner functions (which have their own implicit `this`, shadowing the outer one).

- In strict mode, at the top level `this` is `undefined`.
  - Before ES6, there was no standard portable name for the global object. In the frontend, you had to explicitly use [`window`](https://www.w3schools.com/js/js_window.asp), and in the Node.js backend, [`global`](https://stackoverflow.com/questions/43627622/what-is-the-global-object-in-nodejs).
  - Fortunately, ES6 added the silly-named [`globalThis`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/globalThis), which should work uniformly in any ES6 environment.
  - OTOH, the main use of the global object - without caring about what it is - is for the pre-ES6 module pattern, to be able to inject exports, and then to access them from other scripts. If you use ES6 modules, this use case vanishes.

- [Hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting) may cause the execution order to differ from the presentation order of the source code.
  - Particularly, a function definition may be available earlier than it appears in the source code.
  - See the [temporal dead zone (TDZ)](https://wsvincent.com/javascript-temporal-dead-zone/) concerning differences between `var` vs. `let` and `const` variables in how they hoist.

- For nested function definitions, *function statements* are only allowed at the top level of the surrounding function, but variables can be defined anywhere.
  - So you can just define a variable and assign a *function expression* to it to define a function anywhere.
  - [Function statements are hoisted](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function) to the beginning of the surrounding function scope (or top-level scope when outside any function). Function expressions are not hoisted.
  - Just like in Python, nesting function definitions [creates closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions).

- Creating an object instance **requires** saying `new Foo(...)`, not just `Foo(...)`, **unlike in Python**; using the `new` operator will set the right `this` for you.
  - Calling the constructor function normally (without the `new`) fails to set `this` to the new object instance, which may smite you much later.
  - The ES6 `class` syntax helps. A constructor of a `class` cannot be called without `new`; ES will emit a helpful `TypeError` when attempting to call a `class` constructor without the `new`.
  - For classical function-based constructors (no `class`) you could:

    ```javascript
    function ctor (f) {  // pythonic decorator: make f into a constructor
        function autonew (...args) {
            return new f(...args);
        }
        return autonew;
    }

    var MyObject = ctor(function (...) {...});
    var mo = MyObject();  // now ok, MyObject is a ctor
    ```

    but it's not idiomatic (and this barebones gist fails to set the name for tracebacks correctly).

- Arrow functions don't have their own `this`. This lack of uniformity is supposedly [a feature](https://tc39wiki.calculist.org/es6/arrow-functions/).
  - You can just use a function expression instead of an arrow function, but for the reasons in the link, an arrow function is sometimes a better fit in practice. Particularly, because an arrow function doesn't have its own `this`, using an arrow function makes it easy to refer to the surrounding context's `this`, without defining a `that`.
  - Somehow this brings to mind how the schizophrenic handling of indentation vs. parentheses in statements vs. expressions in Python is supposedly [a feature](https://www.artima.com/weblogs/viewpost.jsp?thread=147358). That's just nonsense, even though it comes from the [BDFL](https://en.wikipedia.org/wiki/Benevolent_dictator_for_life). As [SRFI-110: Sweet expressions](https://srfi.schemers.org/srfi-110/srfi-110.html) shows, a consistent uniform approach is possible, with indentation and parentheses acting as alternative notation for structure *regardless of context*.


### Half of the standard library missing?

Yes, literally. Even some of the Python builtins, though the language features to support them exist.

To get you started, here are custom ES implementations of pythonic `all`, `any`, `range`, `zip` and `enumerate`:

```javascript
function all (predicate, iterable) {  // Same semantics as in Python.
    for (const x of iterable)
        if (!predicate(x))
            return false;
    return true;
}

function any (predicate, iterable) {  // Same semantics as in Python.
    for (const x of iterable)
        if (predicate(x))
            return true;
    return false;
}

// Unlike Python, we allow Infinity, so this also provides the equivalent of `itertools.count`.
// function* range(stop, {start=0, step=1} = {}) {  // if you want to emulate kwargs.
function* range (...args) {  // lazy, like in Python.
    var start=0, stop=Infinity, step=+1;
    if (args.length === 1)
        [stop] = args;
    else if (args.length === 2)
        [start, stop] = args;
    else if (args.length === 3)
        [start, stop, step] = args;
    else if (args.length > 3)
        throw new TypeError("range: invalid call signature. Usage: range(), range(stop), range(start, stop) or range(start, stop, step).")
    if (step === 0)
        throw new RangeError("range: step must be nonzero.")
    var cmp = (step > 0) ? ((a, b) => (a < b)) : ((a, b) => (a > b));
    for (let k = start; cmp(k, stop); k += step)
        yield k;
}

function iter (iterable) {  // FP API, like in Python
    const iterator = iterable[Symbol.iterator]();
    return iterator;
}
function next (iterator) {  // FP API, like in Python
    return iterator.next();
}
function* zip (...xss) {  // zip(as, bs, ...) --> [[a[0], b[0], ...], [a[1], b[1], ...], ...]
    var its = xss.map(xs => iter(xs));
    while (true) {
        let vals = its.map(it => next(it));
        // Unlike Python, which raises StopIteration when the iterable runs out,
        // ES sets the `done` attribute of the object returned by `next`.
        if (any((v => v.done), vals))
            return;
        yield vals.map((v => v.value), vals);
    }
}

function* enumerate (iterable, start=0) {
    let k = start;
    for (let x of iterable) {
        yield [k, x];
        k++;
    }
}
```

I have included `iter` and `next` for completeness, because defining a pythonic FP API for those makes the `zip` implementation read more naturally for pythonistas. To build stuff like this, see e.g. the SO posts [[1]](https://stackoverflow.com/questions/50511398/javascript-es6-named-parameters-and-default-values), [[2]](
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators).


#### "Symbol"? As in Lisp symbols?

Well, yes and no. The attentive reader may have noticed something called `Symbol` being used in `iter`.

The natural habitat of the symbol concept is the Lisp family, but as Paul Graham once famously said, [Python effectively has a symbol data type but no syntax for it](http://www.paulgraham.com/diff.html). The interpreter interns some short strings, so e.g. in CPython 3.6 `"foo" is "foo"` evaluates to `True`, despite the appearance that these are two different instances of the string `"foo"`. The defining properties of symbols are that (1) a symbol is equal only to itself, and (2) to check whether two given symbols are equal, a pointer comparison (i.e. comparing object identity) is sufficient. This is achieved by [interning](https://en.wikipedia.org/wiki/String_interning) the string representation. So it can be argued that in Python, `"foo"` is a symbol. You just have to be aware of the rules which strings get interned; and since in Python this is considered a language implementation detail, you can't rely on it across versions or different interpreters (e.g. PyPy3).

In contrast, in Lisp, `'foo` is a symbol. Interning is guaranteed. It is different from the string `"foo"`, for which no such guarantee is made.

Now, if you have some Lisp background, **beware of the name**. In ES, constructing a new `Symbol('foo')` does the job of Lisp's `(gensym 'foo)`; it **creates a new unique symbol** which is guaranteed not to conflict with any previous return value of `Symbol`. To be able to refer to an existing symbol, the only way (just like with [`gensym`](http://clhs.lisp.se/Body/f_gensym.htm)) is to stash the return value of the `Symbol` constructor when you create that symbol. That return value **is** your symbol. **By design**, there's no other way to refer to *a gensym* (i.e., a symbol produced by `gensym`; or in ES terminology, by the `Symbol` constructor).

The ES standard predefines some symbols, and stashes them as properties on the `Symbol` object. Particularly, the symbol `Symbol.iterator` is effectively the name of the ES equivalent of Python's `__iter__` method - without causing any name conflicts, no matter what names are used for any other properties of the object. Nothing except `Symbol.iterator` itself compares equal to `Symbol.iterator`, so no name can conflict with it.

When referring to properties named with a symbol, the square bracket notation comes in useful, because **by design**, a gensym **has no textual representation**. There is no string name you could place in the source code to refer to a gensym. A gensym may have a human-readable string representation that is printed when you print it, but that string is not its name; it cannot be used as an identifier to refer to it. (This is a feature; if it could, guaranteeing the absence of name conflicts between gensyms would be difficult or impossible.)

So in the `iter` implementation above, `iterable[Symbol.iterator]` retrieves the ES equivalent of the `__iter__` method of the object `iterable`, and then the `()` just calls the method.


### Getting libraries

#### Frontend

Just google for libraries. Plonk the library files where the browser can find them (in the context of your app), and tell your HTML to load them using `<script>` tags.

Because in the browser, *the theoretically simple becomes a legacy-driven quirk-hole* (Jake Archibald), to get the load order of your `.js` files right, see [Deep dive into the murky waters of script loading](https://www.html5rocks.com/en/tutorials/speed/script-loading/), and the SO questions  [[1]](https://stackoverflow.com/questions/436411/where-should-i-put-script-tags-in-html-markup), [[2]](https://stackoverflow.com/questions/4659409/implications-of-multiple-script-tags-in-html). Also a look at [various onload events](https://javascript.info/onload-ondomcontentloaded) may be useful.

You can also use the [ES6 module system](https://www.ecma-international.org/ecma-262/#sec-ecmascript-language-scripts-and-modules) and import the module in your `.js`, if the libraries you're using support ES6 modules and your app uses them.

#### Backend

The backend engine [Node.js](https://nodejs.org/en/) has its own module system that predates ES6. [NPM](https://www.npmjs.com/) is the Node.js equivalent of [PyPI](https://pypi.org/). There's a package manager, `npm`.

If you want to install Node.js-based tools (such as `eslint` or `tern`) globally (not just under a specific project), you'll want to set up a `package.json` to appease the spirits, even though your tool installation directory is obviously not an NPM package. Use [`npm init`](https://docs.npmjs.com/creating-a-package-json-file) to run a questionnaire that will generate the file for you. It will produce something like this:

```json
{
  "name": "mydummypackage",
  "version": "1.0.0",
  "description": "Default package for npm installs",
  "main": "index.js",
  "dependencies": {
  },
  "devDependencies": {
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

You may also want to look into the [`npx`](https://blog.npmjs.org/post/162869356040/introducing-npx-an-npm-package-runner) runner utility.

But if you don't need that many Node.js-based tools, the classic manual solution `ln -s .../node_modules/sometool/bin/sometool ~/.local/bin/sometool` is also fine.


### Packaging and distribution notes

Things you may see in the wild.

Libraries often come in *minified* form (`.min.js`), using one-character identifiers and no whitespace. It's still ES, just not human-readable. This optimizes download size; obfuscation is a side effect. There are [several minifiers available](https://blog.logrocket.com/uglify-vs-babel-minify-vs-terser-a-mini-battle-royale/).

Debuggers often support a *map file* that maps locations in the minified file to the original source code. Alternatively, when testing locally, you can just use the regular non-minified version of the library.

Beside minification, [*tree shaking*](https://en.wikipedia.org/wiki/Tree_shaking), seen e.g. in [Webpack](https://webpack.js.org/) and in [rollup.js](https://rollupjs.org/guide/en/), is a commonly employed optimization technique for optimizing download size. It bundles only the code that a static analysis indicates can actually be reached (when starting from the initial entry point of the program).

[Babel](https://babeljs.io/) is an ES [transpiler](https://en.wikipedia.org/wiki/Source-to-source_compiler) that takes in recent ("next-gen") ES, and produces equivalent older ES, compatible with older browsers.


### Final remarks

Compatibility between different environments providing ECMAScript:

 - Frontend: [Kangax's compatibility table](https://kangax.github.io/compat-table/es2016plus/).
 - Backend: [Node.js compatibility table](https://node.green/).
 - Check an individual feature: [caniuse.com](https://caniuse.com/).

As a rule of thumb for modern ES features in frontend work, **Internet Explorer supports essentially nothing**. All other browsers (even Edge), almost everything. Chrome and Opera are the best, with Firefox not too far behind. So for the frontend, to help yourself stay sane, just require your users to use **anything** except IE. Even corporate-minded environments should have at least Edge by now. (And everyone else already has Firefox or Chrome.)

If you ♡:

 - [FP](https://en.wikipedia.org/wiki/Functional_programming): [Ramda](https://ramdajs.com/), a practical functional library.
 - advanced math: [LALOLib](https://mlweb.loria.fr/lalolab/lalolib.html). E.g. solve eigenvalue problems using nothing but ES. Ideal for the frontend.
 - data visualization (plotting): [D3.js](https://d3js.org/).
 - [syntactic macros](https://en.wikipedia.org/wiki/Macro_(computer_science)#Syntactic_macros): [Sweet.js](https://www.sweetjs.org/), the macro compiler for ES.
   - Unlike [MacroPy](https://macropy3.readthedocs.io/en/latest/), Sweet.js requires a separate compile step, because ES has no [load-time hook](https://docs.python.org/3/reference/import.html#import-hooks) to plug a macro expander in to.

Decent development tools are essential:

 - [ESLint](https://eslint.org/), the static analyzer. Catch potential bugs and style problems before you run.
 - [Tern.js](https://ternjs.net/), the smart autocompleter. It's a plugin supported by many editors/IDEs.
 - A decent editor/IDE, such as [Spacemacs](https://www.spacemacs.org/), the beefed-up Emacs, with its `javascript` layer.
   - The layer uses `js2-mode` as the IDE. It will automatically use ESLint and Tern.js if available.
   - Plugins such as [smartparens](https://github.com/Fuco1/smartparens), [rainbow-delimiters](https://github.com/Fanael/rainbow-delimiters) and [highlight-parentheses](https://github.com/tsdh/highlight-parentheses.el) really make a difference in a paren-heavy language (i.e. almost everything except Python).
   - Emacs (in any variant, starting with version 24.4) also lets you **prettify your view** of the code, **without actually changing** the content of the source file you're editing. For example, if you find concise notation efficient, as in math, and want the ES keyword `function` to be **displayed** as `λ`, that's possible. See [prettify-symbols-mode](https://emacsredux.com/blog/2014/08/25/a-peek-at-emacs-24-dot-4-prettify-symbols-mode/).  For reference, [here's my config](https://github.com/Technologicat/spacemacs.d/blob/master/prettify-symbols-config.el).
   - Obviously, there are other IDEs and I just happen to prefer a modern Emacs; your mileage may vary.
 - Some major web browsers come with a frontend debugger.
   - [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools).
   - [Firefox Developer Tools](https://developer.mozilla.org/en-US/docs/Tools).

As for backend debuggers, you'll have to search for them yourself. I've debugged the backend the same way I debug Python, with copious amounts of `console.log` (the ES equivalent of Python's `print`).

That's all, folks!

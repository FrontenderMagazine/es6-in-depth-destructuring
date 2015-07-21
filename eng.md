# ES6 In Depth: Destructuring

_[ES6 In Depth][1] is a series on new features being added to the JavaScript programming language in the 6th Edition of the ECMAScript standard, ES6 for short._

_Editor’s note: An earlier version of today’s post, by Firefox Developer Tools engineer [Nick Fitzgerald][2], originally appeared on Nick’s blog as [Destructuring Assignment in ES6][3]._

## What is destructuring assignment?

Destructuring assignment allows you to assign the properties of an array or object to variables using syntax that looks similar to array or object literals. This syntax can be extremely terse, while still exhibiting more clarity than the traditional property access.

Without destructuring assignment, you might access the first three items in an array like this:

    var first = someArray[0];
    var second = someArray[1];
    var third = someArray[2];

With destructuring assignment, the equivalent code becomes more concise and readable:

    var [first, second, third] = someArray;

SpiderMonkey (Firefox’s JavaScript engine) already has support for most of destructuring, but not quite all of it. [Track SpiderMonkey’s destructuring (and general ES6) support in bug 694100][4].

## Destructuring arrays and iterables

We already saw one example of destructuring assignment on an array above. The general form of the syntax is:

    [ variable1, variable2, ..., variableN ] = array;

This will just assign variable1 through variableN to the corresponding item in the array. If you want to declare your variables at the same time, you can add a `var`, `let`, or `const` in front of the assignment:

    var [ variable1, variable2, ..., variableN ] = array;
    let [ variable1, variable2, ..., variableN ] = array;
    const [ variable1, variable2, ..., variableN ] = array;

In fact, `variable` is a misnomer since you can nest patterns as deep as you would like:

    var [foo, [[bar], baz]] = [1, [[2], 3]];
    console.log(foo);
    // 1
    console.log(bar);
    // 2
    console.log(baz);
    // 3

`

Furthermore, you can skip over items in the array being destructured:

`

    var [,,third] = ["foo", "bar", "baz"];
    console.log(third);
    // "baz"

And you can capture all trailing items in an array with a “rest” pattern:

    var [head, ...tail] = [1, 2, 3, 4];
    console.log(tail);
    // [2, 3, 4]

When you access items in the array that are out of bounds or don’t exist, you get the same result you would by indexing: `undefined`.

    console.log([][0]);
    // undefined

    var [missing] = [];
    console.log(missing);
    // undefined

Note that destructuring assignment with an array assignment pattern also works for any iterable:

    function* fibs() {
      var a = 0;
      var b = 1;
      while (true) {
        yield a;
        [a, b] = [b, a + b];
      }
    }

    var [first, second, third, fourth, fifth, sixth] = fibs();
    console.log(sixth);
    // 5

## Destructuring objects

Destructuring on objects lets you bind variables to different properties of an object. You specify the property being bound, followed by the variable you are binding its value to.

    var robotA = { name: "Bender" };
    var robotB = { name: "Flexo" };

    var { name: nameA } = robotA;
    var { name: nameB } = robotB;

    console.log(nameA);
    // "Bender"
    console.log(nameB);
    // "Flexo"

There is a helpful syntactical shortcut for when the property and variable names are the same:

    var { foo, bar } = { foo: "lorem", bar: "ipsum" };
    console.log(foo);
    // "lorem"
    console.log(bar);
    // "ipsum"

And just like destructuring on arrays, you can nest and combine further destructuring:

    var complicatedObj = {
      arrayProp: [
        "Zapp",
        { second: "Brannigan" }
      ]
    };

    var { arrayProp: [first, { second }] } = complicatedObj;

    console.log(first);
    // "Zapp"
    console.log(second);
    // "Brannigan"

When you destructure on properties that are not defined, you get `undefined`:

    var { missing } = {};
    console.log(missing);
    // undefined

One potential gotcha you should be aware of is when you are using destructuring on an object to assign variables, but not to declare them (when there is no `let`, `const`, or `var`):

    { blowUp } = { blowUp: 10 };
    // Syntax error

This happens because the JavaScript grammar tells the engine to parse any statement starting with `{` as a block statement (for example, `{ console }` is a valid block statement). The solution is to either wrap the whole expression in parentheses:

    ({ safe } = {});
    // No errors

## Destructuring values that are not an object, array, or iterable

When you try to use destructuring on `null` or `undefined`, you get a type error:

    var {blowUp} = null;
    // TypeError: null has no properties

However, you can destructure on other primitive types such as booleans, numbers, and strings, and get `undefined`:

    var {wtf} = NaN;
    console.log(wtf);
    // undefined

This may come unexpected, but upon further examination the reason turns out to be simple. When using an object assignment pattern, the value being destructured is [required to be coercible to an `Object`][5]. Most types can be converted to an object, but `null` and `undefined` may not be converted. When using an array assignment pattern, the value must [have an iterator][6].

## Default values

You can also provide default values for when the property you are destructuring is not defined:

    var [missing = true] = [];
    console.log(missing);
    // true

    var { message: msg = "Something went wrong" } = {};
    console.log(msg);
    // "Something went wrong"

    var { x = 3 } = {};
    console.log(x);
    // 3

_(Editor’s note: This feature is currently implemented in Firefox only for the first two cases, not the third. See [bug 932080][7].)_

## Practical applications of destructuring

### Function parameter definitions

As developers, we can often expose more ergonomic APIs by accepting a single object with multiple properties as a parameter instead of forcing our API consumers to remember the order of many individual parameters. We can use destructuring to avoid repeating this single parameter object whenever we want to reference one of its properties:

    function removeBreakpoint({ url, line, column }) {
      // ...
    }

This is a simplified snippet of real world code from the Firefox DevTools JavaScript debugger (which is also implemented in JavaScript—yo dawg). We have found this pattern particularly pleasing.

### Configuration object parameters

Expanding on the previous example, we can also give default values to the properties of the objects we are destructuring. This is particularly helpful when we have an object that is meant to provide configuration and many of the object’s properties already have sensible defaults. For example, jQuery’s `ajax` function takes a configuration object as its second parameter, and could be rewritten like this:

    jQuery.ajax = function (url, {
      async = true,
      beforeSend = noop,
      cache = true,
      complete = noop,
      crossDomain = false,
      global = true,
      // ... more config
    }) {
      // ... do stuff
    };

This avoids repeating `var foo = config.foo || theDefaultFoo;` for each property of the configuration object.

_(Editor’s note: Unfortunately, default values within object shorthand syntax still aren’t implemented in Firefox. I know, we’ve had several paragraphs to work on it since that earlier note. See [bug 932080][7] for the latest updates.)_

### With the ES6 iteration protocol

[ECMAScript 6 also defines an iteration protocol][8], which we talked about earlier in this series. When you iterate over [`Map`s (an ES6 addition to the standard library)][9], you get a series of `[key, value]` pairs. We can destructure this pair to get easy access to both the key and the value:

    var map = new Map();
    map.set(window, "the global");
    map.set(document, "the document");

    for (var [key, value] of map) {
      console.log(key + " is " + value);
    }
    // "[object Window] is the global"
    // "[object HTMLDocument] is the document"

Iterate over only the keys:

    for (var [key] of map) {
      // ...
    }

Or iterate over only the values:

    for (var [,value] of map) {
      // ...
    }

### Multiple return values

Although multiple return values aren’t baked into the language proper, they don’t need to be because you can return an array and destructure the result:

    function returnMultipleValues() {
      return [1, 2];
    }
    var [foo, bar] = returnMultipleValues();

Alternatively, you can use an object as the container and name the returned values:

    function returnMultipleValues() {
      return {
        foo: 1,
        bar: 2
      };
    }
    var { foo, bar } = returnMultipleValues();

Both of these patterns end up much better than holding onto the temporary container:

    function returnMultipleValues() {
      return {
        foo: 1,
        bar: 2
      };
    }
    var temp = returnMultipleValues();
    var foo = temp.foo;
    var bar = temp.bar;

Or using continuation passing style:

    function returnMultipleValues(k) {
      k(1, 2);
    }
    returnMultipleValues((foo, bar) => ...);

### Importing names from a CommonJS module

Not using ES6 modules yet? Still using CommonJS modules? No problem! When importing some CommonJS module X, it is fairly common that module X exports more functions than you actually intend to use. With destructuring, you can be explicit about which parts of a given module you’d like to use and avoid cluttering your namespace:

    const { SourceMapConsumer, SourceNode } = require("source-map");

(And if you do use ES6 modules, you know that a similar syntax is available in `import` declarations.)

## Conclusion

So, as you can see, destructuring is useful in many individually small cases. At Mozilla we’ve had a lot of experience with it. Lars Hansen introduced JS destructuring in Opera ten years ago, and Brendan Eich added support to Firefox a bit later. It shipped in Firefox 2\. So we know that destructuring sneaks into your everyday use of the language, quietly making your code a bit shorter and cleaner all over the place.

Five weeks ago, we said that ES6 would change the way you write JavaScript. It is this sort of feature we had particularly in mind: simple improvements that can be learned one at a time. Taken together, they will end up affecting every project you work on. Revolution by way of evolution.

Updating destructuring to comply with ES6 has been a team effort. Special thanks to Tooru Fujisawa (arai) and Arpad Borsos (Swatinem) for their excellent contributions.

Support for destructuring is under development for Chrome, and other browsers will undoubtedly add support in time. For now, you’ll need to use [Babel][10] or [Traceur][11] if you want to use destructuring on the Web.

* * *

_Thanks again to Nick Fitzgerald for this week’s post._

_Next week, we’ll cover a feature that is nothing more or less than a slightly shorter way to write something JS already has—something that has been one of the fundamental building blocks of the language all along. Will you care? Is slightly shorter syntax something you can get excited about? I confidently predict the answer is yes, but don’t take my word for it. Join us next week and find out, as we look at ES6 arrow functions in depth._

_Jason Orendorff_

_[ES6 In Depth][1] editor_

 [1]: https://hacks.mozilla.org/category/es6-in-depth/
 [2]: http://fitzgeraldnick.com/
 [3]: http://fitzgeraldnick.com/weblog/50/
 [4]: https://bugzilla.mozilla.org/show_bug.cgi?id=694100
 [5]: https://people.mozilla.org/~jorendorff/es6-draft.html#sec-requireobjectcoercible
 [6]: https://people.mozilla.org/~jorendorff/es6-draft.html#sec-getiterator
 [7]: https://bugzilla.mozilla.org/show_bug.cgi?id=932080
 [8]: https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/
 [9]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map
 [10]: http://babeljs.io/
 [11]: https://github.com/google/traceur-compiler#what-is-traceur
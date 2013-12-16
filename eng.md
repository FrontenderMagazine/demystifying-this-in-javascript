*By [Nicolas Bevacqua][1]*

In this post I want to shed some light on `this` in JavaScript hopefully bring
some clarity to how`this` works. It’s not all dark magic, learning about `this`
*tremendously helpful* to your development as a JavaScript programmer. The
inspiration for this post comes from my recent work on the latest chapter for my
upcoming book on[JavaScript Application Design][2] (note: you can purchase the
[early access edition][3] now) in which I’m writing about how scoping works.

Until you *“get it”*, this is probably how you feel about `this`:

![chaos][4]

It’s madness, right? In this brief article, I aim to demystify it.

## How this works {#howthisworks}

If a method is invoked on an object, that object will be assigned to `this`.

    var parent = {
        method: function () {
            console.log(this);
        }
    };
    
    parent.method();
    // <- parent

Note that this behavior is very *“fragile”*, if you get a reference to a method
and invoke that, then`this` won’t be `parent` anymore but rather the `window`
global object once again. This confuses most developers.

    var parentless = parent.method;
    
    parentless();
    // <- Window

The bottom line is you should look at the call site to figure out whether the
function is invoked as a property of an object or on its own. If its invoked as 
a property, then that property will become`this`, otherwise `this` will be
assigned the value of the global object, or`window`. In this case, but under 
[strict mode][5], `this` will be `undefined` instead.

In the case of constructor functions, `this` is assigned to the instance that
’s being created, when using the`new` keyword.

    function ThisClownCar () {
      console.log(this);
    }
    
    new ThisClownCar();
    // <- ThisClownCar {}

Note that this behavior doesn’t have a way of telling whether a function is
supposed to be used as a constructor function, and thus omitting the`new`
keyword will result in`this` being the global object, like we saw in the 
`parentless` example.

    ThisClownCar();
    // <- Window

## Tampering with this {#tamperingwiththis}

The `.call`, `.apply`, and `.bind` methods are used to manipulate function
invocations, helping us to define both the value for`this`, and the `arguments`

`Function.prototype.call` takes any number of arguments, the first one is
assigned to`this`, and the rest are passed as arguments to the function that’
s being invoked.

    Array.prototype.slice.call([1, 2, 3], 1, 2)
    // <- [2]

`Function.prototype.apply` behaves very similarly to `.call`, but it takes the
arguments as a single array with every value, instead of any number of parameter
values.

    String.prototype.split.apply('13.12.02', ['.'])
    // <- ['13', '12', '02']

`Function.prototype.bind` creates a special function which can be used to
invoke the function it is called on. That function will always use the`this`
argument passed to`.bind`, as well as being able to assign a few arguments,
creating a[curried version][6] of the original function.

    var arr = [1, 2];
    var add = Array.prototype.push.bind(arr, 3);
    
    // effectively the same as arr.push(3)
    add();
    
    // effectively the same as arr.push(3, 4)
    add(4);
    
    console.log(arr);
    // <- [1, 2, 3, 3, 4]

## Scoping this {#scopingthis}

In the next case, `this` will stay the same across the scope chain. This is the
exception to the rule, and often leads to confusion among amateur developers.

    function scoping () {
      console.log(this);
    
      return function () {
        console.log(this);
      };
    }
    
    scoping()();
    // <- Window
    // <- Window

A common work-around is to create a local variable which holds onto the
reference to`this`, and isn’t shadowed in the child scope. The child scope
shadows`this`, making it impossible to access a reference to the parent `this`
directly.

    function retaining () {
      var self = this;
    
      return function () {
        console.log(self);
      };
    }
    
    retaining()();
    // <- Window

Unless you really want to use both the parent scope’s `this`, as well as the
current value of`this` for some obscure reason, the method I prefer is to use
the`.bind` function. This can be used to assign the parent `this` to the child
scope.

    function bound () {
      return function () {
        console.log(this);
      }.bind(this);
    }
    
    bound()();
    // <- Window

## Other Issues?

Have you ever had any problems with this? How about `this`? Let me know if you
think I’ve missed any other edge cases or elegant solutions.

If you liked this post, be sure to check out my upcoming book 
[JavaScript Application Design: A Build First Approach][2] which you can
already purchase as an[early access edition][3].

*This article was originally published at 
<http://blog.ponyfoo.com/2013/12/04/where-does-this-keyword-come-from>*

 [1]: http://flippinawesome.org/authors/nicolas-bevacqua

 [2]: http://bevacqua.io/buildfirst "JavaScript Application Design: A Build First Approach"
 [3]: http://bevacqua.io/bf/book
 [4]: img/chaos.gif

 [5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode "Strict mode explained on MDN"
 [6]: http://en.wikipedia.org/wiki/Currying "Currying on Wikipedia"
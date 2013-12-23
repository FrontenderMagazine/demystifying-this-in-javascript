Раскрытие тайны this в JavaScript
------------------------------------------------------------

В этой статье я хочу пролить свет на использование `this` в JavaScript и привнести немного ясности и понимание о том, как `this` работает. Это знание не относится к чёрной магии и не является запретным, наоборот — понимание `this` чрезвычайно полезно для вас, как JavaScript-разработчика. Вдохновение для статьи я черпаю из последней главы из моей будущей книги [Архитектура приложения на JavaScript][2] (вы можете заказать [раннее издание][3] сейчас) в которой я описываю как работают зоны видимости.

Пока вы до конца не **осознаете** `this`, вы скорее всего будете чувствовать себя так:

![Хаос][4]

Это безумие, верно? В этой короткой статье я постараюсь прояснить это.

## Как это работает

Если метод был вызван из объекта, тогда `this` в контексте метода будет ссылкой на родительский объект.

    var parent = {
        method: function () {
            console.log(this);
        }
    };
    
    parent.method(); // <- родительский объект

Заметьте, что этот паттерн очень уязвимый, если вы вызовете метод по ссылке, тогда `this` не будет больше ссылаться на родительский объект `parent`, вместо этого `this` теперь будет ссылаться на глобальный объект `Window`. Это обстоятельство приводит в замешательство большинство разработчиков.

    var parentless = parent.method;
    
    parentless(); // <- Window

В последней строчке примера вы должны обратить внимание на то, как вызывается функция: как свойство объекта или сама по себе. Если вызывается как свойство, то `this` будет ссылаться на него (!!!), иначе — на глобальный объект; в данном случае это `Window`, но в [strict mode][5] `this` будет возвращать `undefined`.

В случае конструктора `this` ссылается на созданный экземпляр при условии использования ключевого слова `new`.

    function ThisClownCar () {
      console.log(this);
    }
    
    new ThisClownCar(); // <- ThisClownCar {}

Стоит заметить сильную неявность поведения `this` в конструкторе: если вы случайно забудете `new`, то `this` опять будет ссылаться на глобальный объект, как мы уже наблюдали в примере с `parentless`.

    ThisClownCar();
    // <- Window

## Tampering with this

The `.call`, `.apply`, and `.bind` methods are used to manipulate function
invocations, helping us to define both the value for `this`, and the `arguments`

`Function.prototype.call` takes any number of arguments, the first one is
assigned to `this`, and the rest are passed as arguments to the function that’
s being invoked.

    Array.prototype.slice.call([1, 2, 3], 1, 2)
    // <- [2]

`Function.prototype.apply` behaves very similarly to `.call`, but it takes the
arguments as a single array with every value, instead of any number of parameter
values.

    String.prototype.split.apply('13.12.02', ['.'])
    // <- ['13', '12', '02']

`Function.prototype.bind` creates a special function which can be used to
invoke the function it is called on. That function will always use the `this`
argument passed to `.bind`, as well as being able to assign a few arguments,
creating a[curried version][6] of the original function.

    var arr = [1, 2];
    var add = Array.prototype.push.bind(arr, 3);
    
    // effectively the same as arr.push(3)
    add();
    
    // effectively the same as arr.push(3, 4)
    add(4);
    
    console.log(arr);
    // <- [1, 2, 3, 3, 4]

## Scoping this

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
reference to `this`, and isn’t shadowed in the child scope. The child scope
shadows `this`, making it impossible to access a reference to the parent `this`
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
current value of `this` for some obscure reason, the method I prefer is to use
the `.bind` function. This can be used to assign the parent `this` to the child
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


[1]: http://flippinawesome.org/authors/nicolas-bevacqua

[2]: http://bevacqua.io/buildfirst "JavaScript Application Design: A Build First Approach"
[3]: http://bevacqua.io/bf/book
[4]: img/chaos.gif

[5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode "Strict mode explained on MDN"
[6]: http://en.wikipedia.org/wiki/Currying "Currying on Wikipedia"

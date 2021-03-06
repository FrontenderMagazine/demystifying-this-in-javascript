# Раскрытие тайны this в JavaScript

В этой статье я хочу пролить свет на использование `this` в JavaScript и 
привнести немного ясности и понимание о том, как `this` работает. Это знание 
не относится к чёрной магии и не является запретным, наоборот — понимание 
механизма работы `this` чрезвычайно полезно для всех JavaScript-разработчиков.
Вдохновение для статьи я почерпнул из последней главы моей будущей книги 
[Архитектура приложения на JavaScript][2], которую вы, кстати, можете 
[заказать уже сейчас][3]. В упомянутой выше главе я в том числе описываю то, 
как работают зоны видимости в JavaScript.

Пока вы до конца не **осознаете** `this`, вы скорее всего будете чувствовать 
себя так:

![Хаос][4]

Это безумие, верно? В этой короткой статье я постараюсь прояснить сложившуюся 
ситуацию.

## Как это работает

Если метод был вызван из объекта, тогда `this` в контексте метода является 
ссылкой на родительский объект.

    var parent = {
        method: function () {
            console.log(this);
        }
    };
    
    parent.method(); // ссылается на родительский объект

Заметьте, что этот паттерн очень уязвим, так как при вызове метода по ссылке 
`this` не будет больше ссылаться на родительский объект `parent`. Вместо
этого, `this` теперь будет ссылаться на глобальный объект `Window`. 
Это обстоятельство приводит в замешательство большинство разработчиков.

    var parentless = parent.method;
    
    parentless(); // ссылается на Window

В последней строчке примера вы должны обратить внимание на то, как именно 
вызывается функция - возможны два варианта. Либо функция вызывается как 
свойство объекта, либо она сама по себе. И если она вызывается как свойство, 
то `this` будет ссылаться на данное свойство, в противном случае — на 
глобальный объект. В данном примере это `Window`, но в [строгом режиме][5] 
`this` будет возвращать `undefined`.

В случае с конструктором `this` ссылается на созданный экземпляр при условии
использования ключевого слова `new`.

    function ThisClownCar () {
      console.log(this);
    }
    
    new ThisClownCar(); // ссылается на ThisClownCar {}

Стоит заметить, что поведение `this` в конструкторе весьма неочевидное: если 
вы случайно забудете дописать `new`, то `this` опять будет ссылаться на 
глобальный объект, как мы уже наблюдали в примере с `parentless`.

    ThisClownCar();
    // ссылается на Window


## Ручное управление над this

Методы `.call`, `.apply`, и `.bind` используются для управляемого вызова 
функций, помогая определять оба передаваемых значения — `this`, он же контекст 
функции, и `arguments`.

Метод `Function.prototype.call` может принимать любое количество аргументов. 
Первый из них будет использоваться как `this`, а оставшиеся будут переданы 
вызываемой функции как аргументы.

    Array.prototype.slice.call([1, 2, 3], 1, 2)
    // ссылается на [2]

Метод `Function.prototype.apply` очень похож на предыдущий, только аргументы в 
нём передаются одним массивом, вместо неопределённого количеств аргументов, 
как в методе `call`.

    String.prototype.split.apply('13.12.02', ['.'])
    // ссылается на ['13', '12', '02']

Метод `Function.prototype.bind` возвращает специальную функцию, которая, в 
свою очередь, может быть использована для вызова еще одной функции. От её 
имени и будет вызван `bind`. Функция всегда будет использовать переданный ей 
`this`, и в тоже время, ей можно передать несколько аргументов, в качестве 
всегда используемых  в возвращаемой функции. Это может быть очень удобно для 
[каррирования][6] оригинальной функции.

    var arr = [1, 2];
    var add = Array.prototype.push.bind(arr, 3);
    
    // эквивалентно arr.push(3)
    add();
    
    // эквивалентно arr.push(3, 4)
    add(4);
    
    console.log(arr);
    // ссылается на [1, 2, 3, 3, 4]


## Область видимости и this

В следующем случае `this` будет неизменным в разных областях видимости. Это
исключение в правиле и оно часто приводит к ошибкам среди начинающих 
разработчиков. 

    function scoping () {
      console.log(this);
    
      return function () {
        console.log(this);
      };
    }
    
    scoping()();
    // ссылается на Window
    // ссылается на Window

Распространённый способ обойти создавшуюся проблему — это создать локальную
переменную, содержащую ссылку на `this` в текущей функции, а значит, и в ее 
области видимости. Она останется доступной во вложенной функции. Вложенная 
функция, в свою очередь, будет иметь свою собственную переменную `this`, что 
означает невозможность использования `this` родительской функции напрямую.

    function retaining () {
      var self = this;
    
      return function () {
        console.log(self);
      };
    }
    
    retaining()();
    // ссылается на Window

Если вы всё-таки по неизвестным мне причинам захотите использовать 
родительский `this` в качестве контекста для вложенной функции, то 
я могу посоветовать вам предпочитаемый лично мною метод с использованием 
функции `.bind`. Эта функция может быть использована в том числе для того, 
чтобы пробросить родительский контекст во вложенную функцию.

    function bound () {
      return function () {
        console.log(this);
      }.bind(this);
    }
    
    bound()();
    // ссылается Window

## Вопросы?

Осталось ли для вас что-то неясное в рамках этой статьи? Стал ли вам хоть 
немного понятней механизм работы `this`? Сообщите мне, если на ваш взгляд  
я пропустил какие-то важные ситуации или красивые решения.

Если вам понравился этот пост, то возможно вам стоит вам присмотреться к моей
будущей книге «[Архитектура приложения на JavaScript][2]». Вы можете заказать 
[раннее издание][3] уже сейчас.

[1]: http://flippinawesome.org/authors/nicolas-bevacqua
[2]: http://bevacqua.io/buildfirst "JavaScript Application Design: A Build First Approach"
[3]: http://bevacqua.io/bf/book
[4]: img/chaos.gif
[5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode "Strict mode explained on MDN"
[6]: http://ru.wikipedia.org/wiki/Каррирование "Каррирование на Wikipedia"

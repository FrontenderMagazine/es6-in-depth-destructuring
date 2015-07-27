# ES6 в деталях: Деструктурирование

_[ES6 в деталях][1] — это цикл статей о новых возможностях языка
программирования JavaScript, появившихся в 6 редакции стандарта ECMAScript,
кратко — ES6._

_Примечание редактора: Более ранняя версия сегодняшней статьи за авторством
разработчика Firefox Developer Tools [Ника Фицджеральда (Nick Fitzgerald)][2]
изначально появилась в блоге Ника под названием
«[Деструктурирующее присваивание в ES6][3]»._

## Что такое деструктурирующее присваивание?

Деструктурирующее присваивание позволяет присваивать переменным свойства
массивов или объектов при помощи синтаксиса, который внешне напоминает
литералы объектов и массивов. Этот синтаксис может быть очень сжатым, но при
этом он более доходчив, чем традиционный доступ к переменным.

Без деструктурирующего присваивания вы можете обратиться к первым трём элементам
массива вот так:

    var first = someArray[0];
    var second = someArray[1];
    var third = someArray[2];

С деструктурирующим присваиванием эквивалентный код становится более лаконичным
и читаемым:

    var [first, second, third] = someArray;

В SpiderMonkey (движок JavaScript в Firefox) уже поддерживается большая часть
того, что связано с деструктурированием, но пока ещё не всё.
[Следить за поддержкой деструктурирования (и ES6 в целом) в SpiderMonkey можно в баге 694100][4].

## Деструктурирование массивов и итерируемых объектов

Мы уже встретили выше один пример того, как деструктурирующее присваивание
работает с массивами. В общем виде синтаксис такой:

    [ переменная1, переменная2, ..., переменнаяN ] = массив;

Это всего лишь присвоит переменным от `переменная1` до `переменнаяN`
соответствующие элементы массива. Если вы хотите в то же время определить
переменные, вы можете добавить перед присваиванием `var`, `let` или `const`:

    var [ переменная1, переменная2, ..., переменнаяN ] = массив;
    let [ переменная1, переменная2, ..., переменнаяN ] = массив;
    const [ переменная1, переменная2, ..., переменнаяN ] = массив;

Вообще, говоря, `переменная` — это не совсем верно, ведь мы можем вкладывать
шаблоны друг в друга так глубоко, как этого захочется:

    var [foo, [[bar], baz]] = [1, [[2], 3]];
    console.log(foo);
    // 1
    console.log(bar);
    // 2
    console.log(baz);
    // 3

Более того, можно пропускать элементы массива при деструктурировании:

    var [,,third] = ["foo", "bar", "baz"];
    console.log(third);
    // "baz"

А ещё можно собрать все элементы в хвосте массива используя «остаточный»
шаблон:

    var [head, ...tail] = [1, 2, 3, 4];
    console.log(tail);
    // [2, 3, 4]

Если вы обращаетесь к элементам массива, которые находятся за границей массива
или не существуют, вы получите тот же результат, что и при обычном обращении —
`undefined`.

    console.log([][0]);
    // undefined

    var [missing] = [];
    console.log(missing);
    // undefined

Обратите внимание, деструктурирующее присваивание с шаблоном массива разботает
также с любым итерируемым объектом:

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

## Деструктурирование объектов

Деструктурирование на объектах позволяет сопоставлять переменным различные
свойства объектов. Вы указываете сопоставляемое свойство, а затем переменную,
с которой оно сопоставляется.

    var robotA = { name: "Бендер" };
    var robotB = { name: "Флексо" };

    var { name: nameA } = robotA;
    var { name: nameB } = robotB;

    console.log(nameA);
    // "Бендер"
    console.log(nameB);
    // "Флексо"

Есть ещё полезная синтаксическая конструкция на случай, когда имена свойства и
переменной совпадают:

    var { foo, bar } = { foo: "lorem", bar: "ipsum" };
    console.log(foo);
    // "lorem"
    console.log(bar);
    // "ipsum"

И точно так же, как и с массивами, деструктурирования можно вкладывать и
совмещать:

    var complicatedObj = {
      arrayProp: [
        "Зепп",
        { second: "Бранниган" }
      ]
    };

    var { arrayProp: [first, { second }] } = complicatedObj;

    console.log(first);
    // "Зепп"
    console.log(second);
    // "Бранниган"

Если при деструктурировании вы обратитесь к свойствам, которые не определены,
вы получите `undefined`:

    var { missing } = {};
    console.log(missing);
    // undefined

С деструктурированием объекта, когда оно используется только для присваивания
переменных, но не для их объявления (т.е., когда нет `let`, `const` или `var`),
может быть связана одна ошибка, и об этом вам следует знать:

    { blowUp } = { blowUp: 10 };
    // Ошибка синтаксиса

Это происходит потому что грамматика JavaScript указывает движку парсить любую
конструкцию, начинающуюся с `{` как блок (например, `{ console }` — это вполне
допустимый блок кода). Решением может стать оборачивание всего выражения в
скобки:

    ({ safe } = {});
    // Нет ошибок

## Деструктурирование значений, не являющихся объектом, массивом или итерируемым

Если вы попробуете деструктурировать `null` или `undefined`, вы получите ошибку
о неподходящем типе:

    var {blowUp} = null;
    // TypeError: у null нет свойств

Однако, вы можете деструктурировать другие примитивные типы, такие как булевы
значения, числа или строки, и вы получите `undefined`:

    var {wtf} = NaN;
    console.log(wtf);
    // undefined

Такое поведение может показаться неожиданным, но после дальнейшего изучения
причина окажется простой. При использовании шаблона деструктурирования,
деструктурируемое значение [должно приводиться к объекту][5]. Большинство типов
могут быть преобразованы в объект, но `null` и `undefined` преобразовать нельзя.
Если вы используете шаблон массива для присваивания, то значение должно
[иметь итератор][6].

## Значения по умолчанию

Вы также можете указать значения по умолчанию на случай, если при свойство,
которое вы хотите деструктурировать, не определено:

    var [missing = true] = [];
    console.log(missing);
    // true

    var { message: msg = "Что-то пошло не так" } = {};
    console.log(msg);
    // "Что-то пошло не так"

    var { x = 3 } = {};
    console.log(x);
    // 3

_(Примечание редактора: Эта функциональность реализована в Firefox только для
первых двух примеров, но не для третьего. См. [баг 932080][7].)_

## Прикладное применение деструктурирования

### Определения параметров функций

Как разработчики мы часто предоставляем более эргономичное API, принимая в
качестве параметра единственный объект с несколькими свойствами вместо того,
чтобы заставлять пользователей нашего API запоминать порядок отдельных
параметров. Мы можем воспользоваться деструктурированием чтобы избежать
повторения этого объекта-параметра всякий раз, как мы хотим обратиться к его
свойству:

    function removeBreakpoint({ url, line, column }) {
      // ...
    }

Это упрощенный пример настоящего, работающего кода из отладчика javaScript в
Firefox DevTools (который в свою очередь сам написан JavaScript. Yo Dawg!)
Нам кажется, что такой подход особенно удобен.

### Параметры с объектами-конфигурациями

Дополняя предыдущий пример, мы также можем задать значения по умолчанию для
свойств деструктурируемых объектов. Это особенно полезно, если у нас есть
объект, представляющий из себя конфигурацию, и для многих из свойств этого
объекта уже есть разумные значения по умолчанию. К примеру, функция `ajax` из
jQuery принимает объект-конфигурацию в качестве второго параметра, и её можно
было бы переписать так:

    jQuery.ajax = function (url, {
      async = true,
      beforeSend = noop,
      cache = true,
      complete = noop,
      crossDomain = false,
      global = true,
      // ... больше настроек
    }) {
      // ... делаем что-то полезное
    };

Это позволяет избежать повторения `var foo = config.foo || theDefaultFoo;` для
каждого свойства объекта-конфигурации в отдельности.

_(Примечание редактора: К сожалению, значения по умолчанию внутри краткой записи
свойств объектов не реализованы в Firefox. Я знаю, у нас было несколько абзацев
с предыдущего примечания, чтобы поработать над этим. Смотрите [баг 932080][7],
чтобы узнать о последних новостях.)_

### Использование с протоколом итераторов из ES6

[ECMAScript 6 также определяет протокол для работы с итераторами][8], о котором
мы уже говорили ранее в этом цикле статей. Когда вы итерируете
[`Map` (дополнение ES6 к стандартной библиотеке)][9], вы получаете набор пар
`[ключ, значение]`. Можно деструктурировать эти пары, чтобы удобнее работать как
с ключом, так и со значением:

    var map = new Map();
    map.set(window, "глобальный объект");
    map.set(document, "документ");

    for (var [key, value] of map) {
      console.log(key + " — это " + value);
    }
    // "[object Window] — это глобальный объект"
    // "[object HTMLDocument] — это документ"

Перебираем только ключи:

    for (var [key] of map) {
      // ...
    }

Или перебираем только значения:

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
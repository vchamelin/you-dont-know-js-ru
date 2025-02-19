# Вы не знаете JS: Область видимости и замыкания
# Глава 5: Замыкание области видимости

Мы добрались до этого места, как я надеюсь, с очень здравым, цельным пониманием того, как работает область видимости.

Мы сосредоточимся на крайне важной, но постоянно ускользающей, *почти мифологичной* части языка: **замыканиях**. Если вы следили за нашей дискуссией о лексической области видимости до этого момента, ее развязка заключается в том, что замыкание будет чрезвычайно, разочаровывающе, практически самоочевидным. *За ширмой фокусника есть человек и мы вот-вот его увидим*. Нет, его зовут не Крокфорд (Crockford) (прим. переводчика: постоянный участник развития языка JavaScript, создатель текстового формата обмена данными JSON)!

Если у вас всё еще есть изводящие вас вопросы о лексической области видимости, то тогда самое время вернуться и просмотреть еще раз главу 2 перед тем как продолжить читать далее.

## Прозрение

Для тех, кто уже мало-мальски получил опыт в JavaScript, но кто, пожалуй, никогда не ухватывал суть замыканий, *понимание замыкания* может выглядеть как особая нирвана и чтобы достичь ее он должен бороться и идти на жертвы.

Я помню, что несколько лет назад у меня были навыки владения JavaScript, но не было представления о том, что такое замыкания. Подсказка о том, что была *та самая другая сторона* языка, та, которая обещала даже больше возможностей чем те, которыми я обладал, дразнила и манила меня. Я помню как читал исходный код ранних фреймворков пытаясь понять как именно они работают. Я помню тот первый раз, когда что-то подобное "шаблону модуля" начало возникать в моих мыслях. Я очень живо помню эти "*ага!*".

Чего я не знал в те времена и что потребовало от меня годы, чтобы понять, и что я надеюсь передать вам теперь — это вот этот секрет: **замыкание повсюду окружает вас в JavaScript, вы всего лишь должны распознать его и воспользоваться им.** Замыкания — это не особый инструмент, для которого вы должны изучить новый синтаксис и шаблоны. Нет, замыкания — это даже не оружие, которые вы должны изучить, чтобы уметь обращаться и подчинять его себе как Люк обучался Силе.

Замыкания получаются в результате написания кода, который исходит из лексической области видимости. Они просто возникают. Вам и в самом деле не нужно намеренно создавать замыкания, чтобы получить их преимущества. Замыкания создаются и используются вами на протяжении всего вашего кода.
 Что вы *упускаете*, так это то, что должен быть подходящий мысленный контекст, чтобы распознавать, пользоваться и получать выигрыш от замыканий по вашему собственному желанию.

Момент просветления будет выглядеть примерно так: **О, замыкания уже встречаются везде в моем коде, теперь я их наконец-то *вижу*.** Понимание замыкания похоже на то, когда Нео впервые увидел Матрицу.

## Суть дела

Хорошо, достаточно гипербол и бесстыдных ссылок на кино.

Вот откровенное определение того, что вам нужно знать, чтобы понимать и распознавать замыкания:

> Замыкание — это когда функция умеет запоминать и имеет доступ к лексической области видимости даже тогда, когда эта функция выполняется вне своей лексической области видимости.

Давайте рассмотрим небольшой код, чтобы проиллюстрировать это определение.

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a ); // 2
	}

	bar();
}

foo();
```

Этот код выглядит знакомым, напоминая о наших обсуждениях вложенных областей видимости. У функции `bar()` есть *доступ* к переменной `a` во внешней окружающей области видимости на основании правил поиска лексической области видимости (в этом случае это поиск ссылок RHS).

Это "замыкание"?

Ну, технически... *возможно*. Но согласно нашему же определению "что вам нужно знать о замыканиях" выше... *не совсем так*. Думаю, что самым точным образом объяснить ссылку в `bar()` на `a` будет с помощью правил поиска в лексической области видимости и эти правила — *единственная* (важная!) **часть** того, что такое замыкание.

С чисто академической точки зрения, про вышеуказанный код можно сказать, что у функции `bar()` есть *замыкание* на область `foo()` (и разумеется, даже на остальные области видимости, к которым у нее есть доступ, такие как глобальная область видимости в нашем случае). Немного другими словами, говорят, что `bar()` перекрывает область видимости `foo()`. Почему? Потому что `bar()` вложен внутри `foo()`. Ясно и просто.

Но, замыкание, определенное подобным путем, не является ни совсем уж *характерным*, ни выглядящим *проверяемым* в этом коде. Мы ясно видим лексическую область видимости, но замыкание остается какой-то мистической двигающейся тенью за кодом.

Тогда давайте посмотрим на код, который представит замыкание во всей красе:

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 2 -- Ого, замыкание только что было раскрыто, мужики!
```

У функции `bar()` есть доступ в лексической области видимости к внутренней области видимости `foo()`. Но в то же время, мы берем `bar()`, саму функцию, и передаем ее *как* значение. В этом случае мы `возвращаем` сам объект функции, на который ссылается `bar`.

После выполнения `foo()` мы присвоим значение, которая она возвращает (нашу внутреннюю функцию `bar()`) в переменную `baz`, а  затем мы фактически  вызовем `baz()`, который конечно вызовет нашу внутреннюю функцию `bar()`, просто с помощью другой ссылки в идентификаторе.

Естественно `bar()` выполнится. Но в этом случае, она выполняется *снаружи* относительно своей объявленной лексической области видимости.

После выполнения `foo()`, обычно мы ожидаем, что вся внутренняя область видимости `foo()` целиком будет удалена, поскольку мы знаем что *Движок* использует *сборщик мусора*, который исследует и освобождает память, которая больше не используется. Поскольку видно, что содержимое `foo()` больше не используется, кажется естественным, что оно должно быть *удалено* сборщиком.

Но "магия" замыканий не даст этому произойти. Эта внутренняя область видимости на самом деле *все еще* "используется", а потому не будет удалена. Кто ее использует? **Сама функция `bar()`**.

Благодаря тому, где она была объявлена, у `bar()` есть замыкание лексической области видимости на внутренную область видимости `foo()`, которая удерживает область видимости для `bar()`, чтобы ссылаться на нее позднее.

**`bar()` все еще содержит ссылку на эту область видимости и эта ссылка называется замыканием.**

Таким образом, несколькими микросекундами позже, когда переменная `baz` вызывается (вызывая внутреннюю функцию, которую мы изначально пометили как `bar`), у нее как и полагается есть *доступ* к лексической области видимости, определенной на этапе разработки, поэтому у нее есть доступ к переменной `a` как мы и ожидали.

Функция вызывается корректно вне ее лексической области видимости, определенной на этапе разработки. **Замыкание** дает функции возможность продлить доступ к лексической области видимости, которая была определена на этапе разработки.

Конечно, любой из различных путей, которыми можно эти функции *передавать* как значения, а следовательно вызывать в других местах, является примером контролирующего замыкания.

```js
function foo() {
	var a = 2;

	function baz() {
		console.log( a ); // 2
	}

	bar( baz );
}

function bar(fn) {
	fn(); // смотри мам, я видел замыкание!
}
```

Мы передаем внутреннюю функцию `baz` в `bar` и вызываем ее (теперь как `fn`), и когда мы это делаем, ее замыкание на внутреннюю область видимости `foo()` соблюдается посредством доступа к `a`.

Такие передачи функций могут быть также и непрямыми.

```js
var fn;

function foo() {
	var a = 2;

	function baz() {
		console.log( a );
	}

	fn = baz; // присваиваем `baz` глобальной переменной
}

function bar() {
	fn(); // смотри мам, я видел замыкание!
}

foo();

bar(); // 2
```

Какую бы возможность мы ни использовали, чтобы *передать* внутреннюю функцию вне ее лексической области видимости, эта возможность будет помогать удерживать ссылку на область видимости там, где она была объявлена изначально и когда бы мы ни выполняли ее это замыкание будет соблюдаться.

## Теперь я вижу

Ранее приводимые части кода были немного академичны и искусственно сконструированы, чтобы проиллюстрировать *использование замыкания*. Но я обещал вам что-то большее, нежели просто новую крутую игрушку. Я обещал, что замыкание будет чем-то, что окружает вас повсюду в вашем коде. Давайте теперь  *взглянем* на эту правду.

```js
function wait(message) {

	setTimeout( function timer(){
		console.log( message );
	}, 1000 );

}

wait( "Привет, замыкание!" );
```

Мы берем внутреннюю функцию (с именем `timer`) и передаем ее в `setTimeout(..)`. Но у `timer` есть замыкание области видимости на область `wait(..)`, разумеется с хранением и использованием ссылки на переменную `message`.

Через тысячу миллисекунд после того, как мы выполнили `wait(..)` и ее внутренняя область видимости, которой в ином случае следовало исчезнуть, у внутренней функции `timer` все еще есть замыкание на ту область видимости.

Глубоко во внутренностях *Движка*, встроенная функция `setTimeout(..)` держит ссылку на некоторый параметр, возможно названный `fn` или `func`, или как-то похоже. *Движок* выполняет эту функцию, которая вызывает нашу внутреннюю функцию `timer`, а ссылка на лексическую область видимости все еще остается целой.

**Замыкание.**

Или если вы из лагеря jQuery (или любого другого похожего JS фреймворка):

```js
function setupBot(name,selector) {
	$( selector ).click( function activator(){
		console.log( "Активирую: " + name );
	} );
}

setupBot( "Closure Bot 1", "#bot_1" );
setupBot( "Closure Bot 2", "#bot_2" );
```

Я не знаю какой код пишете вы, но я регулярно пишу код, который ответственен за управление всей глобальной армией ботов замыканий, так что это абсолютно правдоподобно!

Если серьезно, фактически *когда бы* и *где бы* вы ни обращались с функциями  (у которой есть доступ к их собственным лексическим областям видимости) как со значениями первого класса и ни передавали их повсюду, вы скорее всего увидите, что эти функции образуют замыкание. Будь это таймеры, обработчики событий, Ajax-запросы, кросс-оконные сообщения, вэб-воркеры или любые другие асинхронные (или синхронные!) задачи, когда вы передаете *колбэк-функцию*, приготовьтесь разбрасывать вокруг замыкания!

**Примечание:** Глава 3 рассказывала о шаблоне IIFE. Несмотря на то, что часто говорят, что IIFE (отдельно) — пример явного замыкания, я немного не соглашусь, исходя из нашего определения, данного выше.

```js
var a = 2;

(function IIFE(){
	console.log( a );
})();
```

Этот код "работает", но не выглядит как точная демонстрация замыкания. Почему? Потому что функция (которую мы назвали тут "IIFE") — не выполняется вне своей лексической области видимости. Она все еще вызывается прямо в той же области видимости, в которой была объявлена (окружающая/глобальная область видимости, которая хранит `a`). `a` обнаруживается через обычный поиск в лексической области видимости, а не через замыкание.

Несмотря на то, что замыкание может технически получаться во время объявления переменных, оно *не* обязательно наблюдаемо явно, а потому, как говорят, *это как звук падающего дерева в лесу, когда рядом никого нет.*

Хотя IIFE *сама по себе* не является примером замыкания, она несомненно создает область видимости, и это один из самых распространенных инструментов,   который мы используем для создания области видимости, которая может быть замкнута. Таким образом, IIFE, разумеется, тесно связана с замыканием, даже несмотря на то, что не производит самих замыканий.

Отложи сейчас эту книгу, дорогой читатель. У меня есть для тебя задача. Открой какой-нибудь свой недавний JavaScript-код. Поищи функции как значения и определи где ты уже используешь замыкания, где ты возможно их раньше не замечал.

Я подожду.

Теперь... ты видишь!

## Циклы + замыкание

Самый распространенный канонический пример, используемый для иллюстрации замыкания, приводят с использованием скромного цикла for.

```js
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

**Примечание:** Линтеры частенько ругаются на то, что вы помещаете функции внутрь циклов, так как ошибки непонимания замыкания — **настолько типичны среди разработчиков**. Мы объясним, как в данном случае сделать правильно, раскрыв всю мощь замыкания. Но эта тонкость часто ускользает от линтеров и они ругаются в любом случае, предполагая что вы *на самом деле* не знаете, что делаете.

Сущность этого кусочка кода в том, что мы обычно *ожидаем* такого поведения, что в итоге будут распечатаны числа "1", "2", .. "5", по одному за раз, по одному в секунду соответственно.

На самом деле, если вы запустите этот код, вы получите вывод числа "6" 5 раз подряд, через односекундные интервалы.

**Да?**

Во-первых, позвольте объяснить откуда взялась цифра `6`. Условие окончания цикла — когда `i`  *не* `<=5`. Первый раз это происходит, когда `i` равно 6. Таким образом, вывод результата отражает конечное значение `i` после завершения цикла.

На самом деле это кажется очевидным при повторном взгляде. Все функции обратного вызова функции timeout запускаются прямо после завершения цикла. Фактически, пока тикают таймеры, даже если это был вызов `setTimeout(.., 0)` в каждой итерации цикла, все эти функции обратного вызова все равно будут запущены строго после выполнения цикла, и как следствие напечатают `6` каждый раз.

Но здесь есть более глубокий вопрос на повестке дня. Чего *не хватает* в нашем коде чтобы в самом деле вести себя так как мы себе это семантически представляли?

А не хватает того, что мы пытаемся *предполагать*, что каждая итерация цикла "захватывает" свою собственную копию `i` во время выполнения итерации. Но, тем путем, которым работает область видимости, все 5 этих функций, хотя они и определяются отдельно в каждой итерации цикла, все они **замыкаются на одну и ту же глобальную разделяемую область видимости**, в которой, фактически, только одна переменная `i`.

В таком свете, *конечно*, все функции разделяют ссылку на одну и ту же `i`. Кое-что в структуре циклов как правило смущает нас мыслями о том, что есть еще что-то сложное в их работе. Это не так. Нет никакой разницы по сравнению с тем, как если бы мы объявили каждый вызов timeout из 5 друг за другом вообще без всякого цикла.

Хорошо, итак вернемся обратно к нашему животрепещущему вопросу. Чего же не хватает? Нам нужна ~~больше колокольчиков на шее у коровы~~ более изолированная область видимости. Точнее говоря, нам нужна новая изолированная область видимости для каждой итерации цикла.

Мы изучили в главе 3, что IIFE создает область видимости объявляя и сразу выполняя функцию.

Давайте попробуем:

```js
for (var i=1; i<=5; i++) {
	(function(){
		setTimeout( function timer(){
			console.log( i );
		}, i*1000 );
	})();
}
```

Это сработает? Попробуйте. Я снова жду.

Я не буду и дальше интриговать вас. **А вот и нет!** Но почему? Теперь у нас очевидно есть еще одна лексическая область видимости. Каждая функция обратного вызова для timeout, разумеется, замкнута на свою собственную область видимости на итерацию, создаваемую явно каждым вызовом IIFE.

Недостаточно иметь изолированную область видимости **если эта область пуста**. Взгляните поближе. Наша IIFE — всего лишь пустая ничего не делающая область видимости. Нужно *что-то* внутри нее, чтобы она стала полезной для нас.

Ей нужна ее собственная переменная, с копией значения `i` в каждой итерации.

```js
for (var i=1; i<=5; i++) {
	(function(){
		var j = i;
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})();
}
```

**Эврика! Работает!**

Небольшая вариация на тему, которую некоторые предпочтут вышеуказанной:

```js
for (var i=1; i<=5; i++) {
	(function(j){
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})( i );
}
```

Конечно, поскольку эти IIFE — всего лишь функции, мы можем передать в них `i` и мы можем назвать передаваемый параметр `j`, если хотим, или даже можем снова назвать его `i`. В любом случае код теперь работает.

Использование IIFE внутри каждой итерации создало новую область видимости для каждой итерации, что дало обратным вызовам функции при вызове timeout  возможность захватывать новую область видимости в каждой итерации, в каждой из которых есть переменная с правильным итерационным значением в ней.

Проблема решена!

### И вновь рассмотрим блочную область видимости

Внимательно изучите наш анализ предыдущего решения. Мы использовали IIFE, чтобы создать новую область видимости на каждую итерацию. Иными словами, нам фактически *необходима* **блочная область видимости** для каждой итерации. Глава 3 открыла для нас объявление через `let`, которая "похищает" блок и объявляет переменную прямо в нем.

**Фактически она превратила блок в область видимости, которую мы можем охватить.** Таким образом, следующий прекрасный код "просто работает":

```js
for (var i=1; i<=5; i++) {
	let j = i; // да-да, блочная область видимости для замыкания!
	setTimeout( function timer(){
		console.log( j );
	}, j*1000 );
}
```

*Но и это еще не всё!* (голосом Якубовича). Есть особое поведение для объявлений `let` в заголовке цикла for. Это поведение определяет, что переменная будет объявлена не один раз для всего цикла, **а для каждой итерации**. И, она, конечно же, будет инициализирована в каждой последующей итерации значением с окончания предыдущей итерации.

```js
for (let i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

Насколько это круто? Блочная область видимости и замыкание работают рука об руку, решая все мировые проблемы. Не знаю насчет вас, но меня это делает счастливым JavaScript-ером.

## Модули

Есть еще паттерны программирования, которые эффективно используют мощь замыканий, но которые внешне не особенно похожи на функции обратного вызова. Давайте рассмотрим самый мощный из них: *модуль*.

```js
function foo() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}
}
```

Как видно из этого кода, здесь нет никакого явного замыкания. У нас просто есть некоторые приватные переменные `something` и `another`, а также парочка внутренних функций `doSomething()` и `doAnother()`, у обеих из которых есть лексическая область видимости (а потому и замыкание!) над внутренней областью `foo()`.

А теперь смотрите:

```js
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

Этот шаблон в JavaScript мы называем *модуль*. Самый распространенный путь реализации шаблона модуля часто называют "Действенный модуль" и это тот вариант, который мы тут и представили.

Давайте изучим некоторые факты об этом коде.

Во-первых, `CoolModule()` — просто функция, но ее *надлежит вызвать* для создания объекта модуля. Без выполнения внешней функции не случится ни создание внутренней области видимости, ни создание замыканий.

Во-вторых, функция `CoolModule()` возвращает объект, выполненный в синтаксисе объектного литерала `{ key: value, ... }`. У объекта, который мы возвращаем, есть ссылки на наши внутренние функции, но *не* на наши внутренние переменные. Мы храним их скрытыми и приватными. Правильнее всего будет думать о возвращаемом значении в виде объекта, как по существу о **публичном API для нашего модуля**.

Это объектное возвращаемое значение в итоге присваивается внешней переменной `foo`, а затем мы можем получить доступ к этим методам в API, например к `foo.doSomething()`.

**Примечание:** Не обязательно возвращать настоящий объект из нашего модуля. Мы могли бы просто вернуть напрямую внутреннюю функцию. jQuery как раз является хорошим тому примером. Идентификаторы `jQuery` и `$` — это публичное API для "модуля" jQuery, но они сами по сути просто функции  (которые сами по себе могут иметь свойства, поскольку все функции являются объектами).

У функций `doSomething()` и `doAnother()` есть замыкание на внутреннюю область видимости "экземпляра" модуля (полученного на самом деле вызовом `CoolModule()`). Когда мы передаем эти функции вне лексической области видимости путем ссылок на свойства объекта, который мы возвращаем, то мы в этот момент фактически получаем условия, при которых замыкание может появиться и выполниться.

Излагая проще, есть два "требования", которые должны выполняться для шаблона модуля:

1. Должна быть внешняя окружающая функция и она должна быть вызвана хотя бы раз (каждый раз создается новый экземпляр модуля).

2. Окружающая функция должна возвращать хотя бы одну внутреннюю функцию, для того, чтобы у этой внутренней функции было замыкание на приватную область видимости и был доступ и/или возможность изменения ее внутреннего состояния.

Объект со свойством-функцией сам по себе *на самом деле* не модуль. Объект, возвращаемый вызовом функции, у которого есть только свойства-данные и ни одной замыкающей функции *фактически* не является модулем, с точки зрения здравого смысла.

Вышеприведенный код показывает автономный конструктор модуля с названием `CoolModule()`, который можно вызвать любое количество раз, каждый раз создавая новый экземпляр модуля. Небольшая вариация этого шаблона — это когда вы заботитесь о том, чтобы был только один экземпляр, что-то вроде  "синглтона":

```js
var foo = (function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

Тут мы превратили нашу модульную функцию в IIFE (см. главу 3), *сразу же* вызвали ее и присвоили возвращаемое ею значение прямо нашему единственному идентификатору `foo`.

Модули — это всего лишь функции, поэтому они могут принимать параметры:

```js
function CoolModule(id) {
	function identify() {
		console.log( id );
	}

	return {
		identify: identify
	};
}

var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );

foo1.identify(); // "foo 1"
foo2.identify(); // "foo 2"
```

Еще одна небольшая, но полнофункциональная вариация модульного шаблона — дать имя объекту, который вы возвращаете как публичное API:

```js
var foo = (function CoolModule(id) {
	function change() {
		// modifying the public API
		publicAPI.identify = identify2;
	}

	function identify1() {
		console.log( id );
	}

	function identify2() {
		console.log( id.toUpperCase() );
	}

	var publicAPI = {
		change: change,
		identify: identify1
	};

	return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

Сохраняя внутреннюю ссылку на объект публичного API внутри экземпляра модуля вы можете менять эту ссылку на модуль **изнутри**, включая добавление и удаление методов, свойств *и* изменение их значений.

### Современные модули

Различные загрузчики/менеджеры модульных зависимостей фактически упаковывают это определение модульного шаблона в дружественное API. Прежде чем начать изучать любую отдельную библиотеку, позвольте мне показать *очень простой* экспериментальный вариант **только в иллюстративных целях**:

```js
var MyModules = (function Manager() {
	var modules = {};

	function define(name, deps, impl) {
		for (var i=0; i<deps.length; i++) {
			deps[i] = modules[deps[i]];
		}
		modules[name] = impl.apply( impl, deps );
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();
```

Ключевая часть этого кода — `modules[name] = impl.apply(impl, deps)`. Это вызов функции-обертки определения для модуля (с передачей внутрь любых зависимостей) и сохранение возвращаемого значения, API модуля, во внутренний список модулей со ссылкой по имени.

И вот как именно я мог бы его использовать для определения нескольких модулей:

```js
MyModules.define( "bar", [], function(){
	function hello(who) {
		return "Let me introduce: " + who;
	}

	return {
		hello: hello
	};
} );

MyModules.define( "foo", ["bar"], function(bar){
	var hungry = "hippo";

	function awesome() {
		console.log( bar.hello( hungry ).toUpperCase() );
	}

	return {
		awesome: awesome
	};
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

Оба модуля "foo" и "bar" объявлены с функцией, которая возвращает публичное API. "foo" даже получает экземпляр "bar" как параметр-зависимость и может его использовать соответствующим образом.

Потратьте немного времени на изучение этих примеров кода, чтобы полностью осознать всю мощь замыканий и потом воспользоваться ими в собственных целях. Ключевой вывод тут в том, что нет никакого особого "волшебства" в модульных менеджерах. Они соответствуют обеим характеристикам шаблона модулей, которые я перечислил выше: вызов обертки определения функции и хранение возвращаемого значения в качестве API для этого модуля.

Иными словами, модули — это просто модули, даже если вы делаете ставку на хорошо знакомый вам инструмент для этого.

### Будущие модули

ES6 добавляет синтаксическую поддержку первого класса для концепции модулей. При загрузке через модульную систему, ES6 обрабатывает файл как отдельный модуль. Каждый модуль может как импортировать другие модули или отдельные члены из API, так и экспортировать свои собственные публичные члены API.

**Примечание:** Модули, основанные на функциях — не статически распознаваемый шаблон (о котором знает компилятор), поэтому их семантика API не учитывается до момента времени выполнения. То есть, вы фактически можете менять API модуля в процессе выполнения кода (см. ранее обсуждение `публичного API`).

В противоположность этому, модульное API ES6 — статическое (API не меняется во время выполнения). Поскольку компилятор *это* знает, он может (и делает!)  проверить во время (загрузки файла и) компиляции, что ссылка на член API импортируемого модуля *действительно существует*. Если ссылка на API не существует, компилятор выдаст "заблаговременную" ошибку на этапе компиляции, вместо того, чтобы  ждать традиционного динамического разрешения ссылок во время выполнения (и ошибок, если есть).

У модулей ES6 **нет** "встраиваемого" формата, они должны определяться в отдельных файлах (по одному на модуль). У браузеров/движков есть "загрузчик модулей" по умолчанию (который может быть переопределен, но это уже далеко за пределами нашего здесь обсуждения), который синхронно загружает файл модуля, когда он импортируется.

Рассмотрим код:

**bar.js**
```js
function hello(who) {
	return "Let me introduce: " + who;
}

export hello;
```

**foo.js**
```js
// импортирует только `hello()` из модуля "bar"
import hello from "bar";

var hungry = "hippo";

function awesome() {
	console.log(
		hello( hungry ).toUpperCase()
	);
}

export awesome;
```

```js
// импортирует модули "foo" и "bar" целиком
module foo from "foo";
module bar from "bar";

console.log(
	bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

**Примечание:** Необходимо создать отдельные файлы **"foo.js"** и **"bar.js"**, с указанным выше содержимым. Затем ваша программа загрузит/импортирует эти модули, чтобы использовать их как показано в третьем фрагменте кода.

`import` импортирует один или более членов API модуля в текущую область видимости, каждый в свою переменную (`hello` в нашем случае). `module` импортирует API модуля целиком в указанную переменную (`foo`, `bar` в нашем случае). `export` экспортирует идентификатор (переменную, функцию) в публичное API текущего модуля. Эти операторы можно использовать в определении модуля столько раз, сколько потребуется.

Содержимое внутри *файла модуля* обрабатывается как если бы оно было заключено в замыкание области видимости, также как и у модулей с функцией-замыканием, рассмотренных ранее.

## Итог

Похоже, что знания о замыкании полны предрассудков и суеверий как загадочный мир, стоящий особняком внутри JavaScript, который могут познать только самые храбрые души. Но на самом деле — это всего лишь стандартный и почти очевидный факт о том, как писать код в среде лексической области видимости, где функции являются значениями и могут свободно передаваться куда угодно.

**Замыкание — это когда функция может запомнить и иметь доступ к своей лексической области видимости даже тогда, когда она вызывается вне своей лексической области видимости.**

Замыкания могут сбить нас с толку, например в циклах, если мы не озаботимся тем, чтобы распознавать их и то как они работают. Но они еще и являются весьма мощным инструментом, дающим доступ к шаблонам, таким как *модули* в их различных формах.

Модули требуют две ключевых характеристики: 1) внешнюю функцию-обертку, которую будут вызывать, чтобы создать закрытую область видимости 2) возвращаемое значение функции-обертки должно включать в себя ссылку на не менее чем одну внутреннюю функцию, у которой потом будет замыкание на внутреннюю область видимости обертки.

Теперь вы сможете заметить замыкания повсюду в своем существующем коде и у нас теперь есть возможность обнаруживать и использовать все их преимущества!

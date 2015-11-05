---
layout: post.hbt
title: "ООП в SuperCollider. Часть первая. Сообщения"
alias: supercollider-oop-part-1
date: 2014-10-24 08:49:44 +0300
author: Evgeny Kozhura
collection: posts
header_image: gifs/apocalypse.now.002.gif
---
Прежде чем продолжить цикл о библиотеке паттернов, решил вернуться немного назад и детально разобрать, как писать объектно-ориентированный код в SuperCollider. И первое, что нужно знать - это то, что реализация ООП в SuperCollider аналогична Smalltalk (а это значит, что если документации по SuperCollider недостаточно для понимания ООП, пригодятся и учебные пособия по Smalltalk).

Итак, все сущности в программе являются объектами - числа, строки, коллекции, функции и т.д. Объекты создаются с помощью классов. Класс определяет поведение объектов и их состояния (с помощью методов и свойств, соответственно). Для понимания классов, часто проводят аналогию с чертежами (blueprints), по которым и создаются экземпляры-объекты. Классы упорядочены иерархически - они наследуют поведение от родительского класса и могут передавать свое поведение дочерним классам (подклассам).

```js
a = Dictionary.new; // создаем объект класса Dictionary
```

Объекты взаимодействуют друг с другом посредством сообщений (message). Возможность принимать то или иное сообщение определяется наличием необходимого метода. Например, объект класса Number принимает сообщение `*`, если реализует метод с таким же именем. При этом, за поведение метода `*` целиком отвечает объект-получатель (receiver), сообщение только определяет желаемую операцию (умножение в данном случае). В этом ключевая особенность. Объекты разных классов могут по-разному реагировать на одно и то же сообщение [^1].

Итак, в SuperCollider есть несколько способов вызова методов объекта. Первый способ - собственно передача сообщения объекту-получателю (message passing). Синтаксис следующий - объект-получатель и сообщение разделены точкой, затем в скобках указан список аргументов.

```js
4.dup(3); // вернет [4, 4, 4]
5.max(6); // вернет 6
5.max(6).dup(5); // цепочка сообщений, вернет [6, 6, 6, 6, 6]
```

Второй способ - в виде бинарной операции. Такая запись является единственной возможной для методов, имена которых выраженны спец символами вроде `+`, `-`, `&`, `>` (логические и математические выражения). Так, например умножение мы запишем как `4 * 5` и не иначе (при попытке выполнить `4.*(5)` мы получим ошибку). Но и другие методы, которые принимают 1 аргумент, можно выразить в виде бинарных операций, добавив к имени двоеточие:

```js
4 dup: 3; // вернет [4, 4, 4]
5 max: 6; // вернет 6
5 do: {arg i; i.post;} // выведет на экран 01234 и вернет 5
5 max: 6 dup: 5; // цепочка сообщений, вернет [6, 6, 6, 6, 6]
```

И наконец, существует так называемая функциональная нотация, потому что синтаксически она похожа на запись вызова функции в большинстве языков программирования. Сперва записывается имя сообщения, потом в скобках - список объектов. При этом, первым элементом является объект-получатель, а остальные элементы передаются в метод получателя как аргументы.

```js
dup(4, 3); // вернет [4, 4, 4]
max(5, 6); // вернет 6
dup(max(5, 6), 5); // цепочка сообщений в виде вложенности вызовов функций
```

Последний способ является синтаксическим сахаром, который позволяет писать программы в функциональном стиле. Несмотря на то, что 5 и 6 являются объектами, которые взаимодействуют друг с другом с помощью сообщения max, код выглядит так, будто бы мы применили функцию max к двум целочисленным значениям 5 и 6.

Рассмотрим некоторые свойства сообщений. Язык можно расширять, дополнять новыми конструкциями с помощью механизма сообщений. Например, привычные управляющие конструкции `if-else`, `while`, `case` в SuperCollider (а ранее, в Smalltalk) реализованы с помощью методов. На примере условной конструкции: получателем сообщения `if` является объект типа `Boolean`, который мы получаем в результате выполнения какого-нибудь логического выражения. Сообщение `if` также передает в метод две функции - первая будет выполнена в случае если выражение вернет `true`, вторая - в противоположном случае, при значении `false`. Таким образом, поведение метода `if` целиком соответствует поведению аналогичной конструкции из других императивных языков программирования.

```js
[true, false].choose.if({"Истина";}, {"Ложь";});
/* в фунциональной нотации */
if ([true, false].choose, {"Истина";}, {"Ложь";});
```

В примерах мы уже видели как сообщения могут образовать цепочку (message chaining). Это становится возможным благодаря свойству методов - любой метод гарантировано возвращает какой-то объект. Если в методе не определено какое именно значение должно быть возвращено, по умолчанию метод возвращает ссылку на объект-получатель. Итак, объект-получатель, обработав сообщение, возвращает объект, которому так же можно послать сообщение и так до самого конца цепочки.

```js
10.rand.max(5).dup.dup(4).flat
```

Математические выражения в SuperCollider записываются с помощью бинарных операций, но привычные правила приоритета операций здесь не сработают. В SuperCollider приоритеты всех бинарных операций равны и для вычислений справедливо правило левой ассоциативности - вычисления выполняются слева направо. Так `3 + 4 * 5` вернет 35, а не 23.

В то же время, приоритетность у бинарных операций ниже, чем у сообщений:

```js
/* метод max будет вызван раньше операции сложения, и выражение вернет 11 */
5 + 5.max(6);
/* max записан как бинарная операция, и по правилу левой ассоциативности выполнится после сложения. выражение вернет 10 */
5 + 5 max: 6;
```

Порядок выполнения можно легко переопределить с помощью скобок (они повышают приоритет операции).

Сообщения могут передаваться объекту в качестве аргументов. Рассмотрим пример:

```js
4.perform([\max, \min].choose, 5);
4.tryPerform(\doIt, {| a | a.postln; });
```

`perform` принимает в качестве первого аргумента сообщение, которое нужно передать объекту-получателю. Если объект не "понимает" сообщение, то есть у него нет соответствующего метода, генерируется исключение doesNotUnderstandError. `tryPerform` действует аналогично `perform`, но если объект не обладает соответствующим методом, сообщение ему не посылается и `tryPerform` возвращает `nil` [^2].

Отдельно стоит упомянуть о передаче аргументов. Количество аргументов и их порядок определены в объявлении метода (вместе с именем метода они составляют его сигнатуру). Но при вызове метода не обязательно следовать сигнатуре. Можно передать меньшее количество аргументов (или большее). Для этого в методе нужно указать значения по умолчанию (для тех случаев, когда соответствующий аргумент можно опустить при вызове метода). В противном случае, такой аргумент будет иметь значение `nil`. Чтобы вызвать метод с произвольным порядком аргументов, в вызове необходимо обратиться к аргументу по его имени.

```js
~func = {|first = 2, second = 4|
    first * second;
};
~func.value;    // 8
~func.value(5); // 20
~func.value(second: 3); // 6
```

---
На этот раз все, в следующем посте перейдем к классам в SuperCollider.

[^1]: В статье о потоках мы видели этот подход в действии. Объекты всех классов, начиная с абстрактного Object и включая классы Number, String, семейство паттерн-классов и т.д., реализуют методы `next`, `embedInStream`, `asStream`, но реализация отличается в зависимости от класса. Таким образом, каждый объект может отвечать на сообщение `embedInStream`.

[^2]: Одно замечание по `tryPerform`. Метод `tryPerform` не увидит методы, добавленные во время выполнения программы (например, добавленные с помощью `addUniqueMethod`). Возможно, так происходит потому, что он проверяет среди методов, определенных в классе на этапе компиляции.
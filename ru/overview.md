---
layout: page
title:  Введение
date:   ср, 04-мар-2020 00:00:00 +0200
categories: ru
---
> Внимание, эта статья находится в процессе создания; ее содержание может (и будет) меняться, пока полностью не удовлетворит автора. А до тех пор автор не ручается за стопроцентную достоверность приведенной информации.

Lisp, кто бы что не говорил, не тарабарщина, Lisp - это божественный, безумно [древний](https://ru.wikipedia.org/wiki/IBM_704) язык! И невероятно простой. Научить лиспу можно даже дошкольника, лишь бы он умел читать и писать (неоднократно проверено). Но вместе с этим из очень простых конструкций этого языка можно строить сложные, непостижимо умные программы.

Otus Lisp состоит из двух частей - виртуальной машины со своим специфичным байткодом и транслятора с языка Lisp в этот байткод. Виртуальная машина является очень маленькой (около 42 килобайт) и быстрой, в то время как сам транслятор в 20 раз больше.

В контексте изложения можно встретить разные именования языка, в приложении к которому идет речь. Важно понимать различие:

* [Lisp](https://ru.wikipedia.org/wiki/%D0%9B%D0%B8%D1%81%D0%BF) - базовый язык собственной персоной. Общие концепции языка, философия и синтаксис присущи для всего семейства потомков (диалектов) Лиспа.
* [Scheme](https://ru.wikipedia.org/wiki/Scheme) - один из самых распространенных диалектов языка Lisp (другим не менее распространенным является Common Lisp, вообще же - десятки их). Все, что помечено как Scheme, является специфическим именно для философии Scheme.
* [R<sup>7</sup>RS](http://www.r7rs.org) - один из используемых стандартов Scheme (их несколько, R<sup>3</sup>RS, R<sup>4</sup>RS, ..., R<sup>n</sup>RS), на котором базируется базовая библиотека Ol. Этот стандарт расширяется целым набором дополнительных [SRFI](http://srfi.schemers.org/).
* И, собственно, сам [Otus Lisp](http://otus-lisp.github.io/). Все помеченное этим именем, либо его часто используемым сокращением "Ol", специфично только и именно для описываемой реализации языка.

Следует указать, что все помеченное как Lisp полностью включается в Scheme, а как Scheme - в R<sup>7</sup>RS. Но не все из R<sup>7</sup>RS включается в Ol, ведь Ol - чисто функциональный диалект (с некоторыми оговорками), в то время как стандарт R<sup>7</sup>RS - мультипарадигменный (подразумевает также и императивные элементы поведения языка). Такие различия будут в обязательном порядке оговорены.

И сразу же первая оговорка: начиная с версии 2.0 в Ol с целью оптимизации использования вычислительных ресурсов в целом классе задач были включены ограниченные мутаторы. Об этом тоже будет идти речь в [соответствующем месте](?ru/mutators).


---
layout: page
title:  Виртуальная машина Otus Lisp
date: 2016-11-28 15:51:55 UTC
categories: ru, olvm
---

Виртуальная машина Ol (Otus Lisp Virtual Machine - olvm) является регистровой (R) виртуальной машиной, использующей для хранения данных динамическую кучу (Heap) под управлением сборщика мусора (Garbage Collector, GC).

Все данные, которыми оперирует olvm, как и в большинстве других языков программирования с динамической типизацией, делятся на два типа - непосредственный (value) тип и ссылочный (reference) тип.

Значения value типа представляют собой такие значения, которыми целиком помещаются в регистре, благодаря чему ними можно очень легко манипулировать (передавать, хранить, обрабатывать). Это небольшие числа, булевые константы #true и #false, пустой список '(), некоторые константы (#empty, #eof, ..).

Данные reference типа - это всегда указатели на объекты, размещаемые в памяти. Иногда полезно знать, что эти объекты выравнены по границе машинного слова.

Объекты reference типа никогда и никуда не передаются по значению, только по ссылке. Объекты value типа никогда не передаются по ссылке, только по значению. Если вам надо передать некий объект именно как по значению, вам придется создать его копию. Если же вам надо сохранить в памяти значение value типа, например для ссылки на него из другого места, то его надо упаковать. Имеется ввиду, что надо создать некий reference объект, в который поместить ваше value (например, вектор единичной длины) и передавать уже этот reference. Автоматической упаковки/распаковки (boxing/unboxing) value данных в olvm не предусмотрено.

Сразу отмечу, что в ol запрещены мутирующие манипуляции над данными, представляющими собой reference значения. Впрочем, работы в этом направлении ведутся и не исключено, что в одной из будущих версий появится такая возможность.

#### Система команд

Систему команд olvm можно разделить на классы по нескольким признакам.

Во-первых, по признаку доступности прикладному программисту - некоторые команды имеют собственное имя и представлены среди списка доступных примитивов (встроенная библиотека "src olvm"); другие же имеют только числовой код и в списке не присутствуют. Команды, которые имеют в языке соответствующее обозначение (например cons, car, cdr) доступны для использования прикладным программистом.

Во-вторых, по функциональному назначению - это деление более сложное и детальное, и именно в разрезе назначения мы сейчас рассмотрим все без исключения команды виртуальной машины. Если у команды есть собственное доступное имя, оно будет сразу приведено.

Итак, команды виртуальной машины делятся на:
 * команды управления контекстом и выполнением
 * команды манипуляции данными
 * примитивные операции языка (примопы)

##### Команды управления контекстом
В эту группу команд входят команды вызова функций, возврата из них, безусловного перехода по адресу, а также взаимодействия с менеджером сопрограмм. Все эти команды переключают контекст выполнения, а значит участвуют в работе планировщика выполнения сопрограмм.

 * **2** GOTO, команда безусловного переключения контекста.
 * **20** APPLY (apply f . args), вызывает функцию с указанными аргументами. Эта команда является примитивом языка lisp и хорошо описана в соответствующей документации по языку.
 * **24** RET, выходит из контекста с возвратом значения.
 * **27** SYS (sys), вызывает функцию [менеджера сопрограмм](?ru/comanager) для текущего контекста. Необходимо заметить, что менеджер сопрограмм не входит в состав виртуальной машины и всегда предоставляется исполняемой средой (программой) - соответственно, если менеджер отсутствует, то вызов команды приведет к программному исключению.
 * **50** RUN, выполнить функцию с возвратом в вызывающий контекст.
 * **17** ARITY-ERROR, вызвать исключение не соответствия количества переданных аргументов количеству принимаемых функцией.
 * семейство команд CLOSE
   * **3** OCLOSE(TCLOS)
   * **4** OCLOSE(TPROC)
   * **6** CLOSE1(TCLOS)
   * **7** CLOSE1(TPROC)

##### Команды управления выполнением
В эту группу входят команды, манипулирующие выполнением программы без смены контекста выполнения. Они включают условные и безусловные переходы (всегда относительно текущего адреса выполнения).

 * **8** JEQ, выполнить переход, если аргументы идентичны в смысле eq? (смотреть примитивы сравнения eq? eqv? equal?)
 * семейство команд JP:
   * **16** JZ, выполнить переход, если аргумент равен value(0)
   * **80** JN, выполнить переход, если аргумент равен #null (пустой список)
   * **144** JE, выполнить переход, если аргумент равен #empty (пустой словарь)
   * **208** JF, выполнить переход, если аргумент равен #false
 * **25** JF2, выполнить переход, если количество переданных аргументов не соответствует заявленному
 * **89** JF2x, вариант JF2, с упаковкой "лишних" аргументов списком в последний. Используется для вызова функций вида (lambda (x . rest) ...)

##### Команды манипулирования данными в регистрах
Этот набор команд предназначен для загрузки данных программы в регистры с целью последующей их обработки

 * семейство LDI:
   * **13** LDE, записывает в регистр #empty (пустой словарь)
   * **77** LDN, записывает в регистр #null (пустой список)
   * **141** LDT, записывает в регистр #true
   * **205** LDF, записывает в регистр #false
 * **1** REFI, загрузка в регистр данных по индексу
 * **9** MOVE, передача данных между регистрами
 * **5** MOV2, двойная передача данных между регистрами

##### Команды, соответствующие примитивам языка
Эта группа примитивов языка, реализованных на уровне виртуальной машины. Хотя некоторые из этих команд и могут быть выражены с помощью других примитивов (например cons через vm:new, car через ref и т.д.), но с целью оптимизации времени выполнения программы они реализованы на машинном языке. Ввиду того, что виртуальная машина и так очень маленькая (от 20 KB в зависимости от платформы) существующее решение выглядит вполне оправданным.

Вот этот список в порядке "высокоуровневости" примитива.

 * **23** (vm:new type ...), конструирует объект заданного типа. Аллокатор.
 * **60** (vm:raw type list), конструирует "сырой" объект - объект, содержащий битовую последовательность. Аллокатор. С помощью этой команды можно, например, собрать свою функцию прямо из байткода. Например, эту же функцию raw можно собрать как `(define raw (raw type-bytecode '(60 4 5 6  24 6)))`
 * **51** (cons a b), конструирует пару. Аллокатор.
 * **52** (car pair), возвращает первый элемент пары. Геттер.
 * **53** (cdr pair), возвращает второй элемент пары. Геттер.
 * **47** (ref obj n), возвращает n-й элемент объекта obj, где obj должен быть reference типа. Геттер.
 Для объектов, состоящих из последовательностей байт (raw объектов, таких как string или byte-vector) возвращается именно байтовый элемент. Для остальных - обычный (например, для tuple).
 Внимание, для обычных и сырых объектов индексация отличается - у обычных она начинается с 1, у сырых с 0.
 * **15** (type obj), возвращает тип объекта. Типов всего 64, вот неполный список:
   *  0, type-fix+ - value число (маленькое), позитивное
   * 32, type-fix- - value число (маленькое), негативное
   * 40, type-int+ - reference число (большое), позитивное
   * 41, type-int- - reference число (большое), негативное
   * 42, type-rational - рациональное число, представляет из себя нормализованную пару чисел числитель/знаменатель, к элементам числа можно обращаться через car/cdr
   * 43, type-complex - комплексное число, представляет из себя пару чисел абсцисса/ордината, к элементам числа также можно обращаться через car/cdr
   *  1, type-pair - пара, создаваемая с помощью cons
   *  2, type-tuple - кортеж
   *  3, type-string - строка
   *  4, type-symbol - символ (напоминаю, что символы в ol полноправные элементы языка)
   * 16, type-bytecode - байткод, то, что выполняет olvm
   * 17, type-proc
   * 18, type-clos
   * 31, type-thread-state
   * 15, type-vector-dispatch
   * 11, type-vector-leaf
   * 19, type-vector-raw
   * 14, type-rlist-node
   * 10, type-rlist-spine
   * 22, type-string-wide
   * 21, type-string-dispatch
   *  5, type-record (deprecated)
   * 24, type-ff
   * 25, type-ff-r
   * 26, type-ff-red
   * 27, type-ff-red-r
   *  8, type-ff-black-leaf
   * 13, type-const
   * 12, type-port
 * **36** (size obj), возвращает размер объекта (для raw объектов возвращается размер в байтах, для остальных - в элементах)
 * **22** (cast obj type), производит преобразование типа объекта. Используется в математической библиотеке для смены знака числа, а также для задания значений портов stdin, stdout, stderr. Cпорная операция, в будущем возможно будет исключена из языка.
 * **45** (set-ref obj n value), заменяет элемент obj в позиции n на value, в полном согласии с функциональной парадигмой программирования возвращает новый объект не изменяя старый
 * **10** (set-ref! obj n value), мутатор, аналог set-ref (experimental support)
 * **54** (eq? a b), производит сравнение элементов a и b, если элементы value-типа, сравниваются их значения; если же reference, то адреса в памяти. За деталями обращаться в [R<sup>5</sup>RS](https://groups.csail.mit.edu/mac/ftpdir/scheme-reports/r5rs-html/r5rs_8.html)
 * **44** (less? a b), производит сравнение элементов a и b, если элементы value-типа, сравниваются их значения; если же reference, то адреса в памяти. За деталями обращаться в [R<sup>5</sup>RS], возвращает true если объект a строго меньше объекта b

##### Математические команды
 * Основная часть команд описана в главе "[internals/numbers](?ru/internals/numbers)"
 * **33** (fxmax) - возвращает максимальное число value типа, название команды еще не стандартизировано
 * **34** (fxmbits) - возвращает количество бит в (fxmax) числе

##### Прочие
 * **61** (clock)
 * **63** (syscall n a b c), интерфейс с функциям операционной системы. В основном нумерация совпадает с таблицей системных вызовов linux 64-bit, вот список функций
   *  0, read
   *  1, write
   *  2, open
   *  3, close
   *  4, stat
   *  5, fstat
   * 16, ioctl (поддерживается только TIOCGETA команда, 19, для проверки терминал ли устройство ввода)
   * 41, socket (если включен дефайн HAS_SOCKETS)
   * 42, connect (если включен дефайн HAS_SOCKETS)
   * 48, shutdown (если включен дефайн HAS_SOCKETS)
   * 49, bind (если включен дефайн HAS_SOCKETS)
   * 50, listen (если включен дефайн HAS_SOCKETS)
   * 43, accept (если включен дефайн HAS_SOCKETS)
   * 23, select (если включен дефайн HAS_SOCKETS)
   * 51, getpeername (если включен дефайн HAS_SOCKETS)
   * 35, nanosleep
   * 59, execve
   * 60, exit
   * 63, uname
   * 78, getdents
   * 96, gettimeofday
   * 98, getrusage, (может отсутствовать в виртуальной машине, выключается в заголовочном файле olvm.h с помощью SYSCALL_GETRUSAGE, на данный момент отсутствует в сборке для Windows)
   * 99, sysinfo, (может отсутствовать в виртуальной машине, выключается в заголовочном файле olvm.h с помощью SYSCALL_SYSINFO, на данный момент отсутствует в сборке для Windows)
   * 157, prctl, (может отсутствовать в виртуальной машине, выключается в заголовочном файле olvm.h с помощью SYSCALL_PRCTL)
   * 62, kill
   * 201, time - в отличие от аналогичного системного вызова умеет не только возвращать время, но и форматировать его в нужный строковый формат
   * 174, dlopen
   * 176, dlclose
   * 177, dlsym
   * 178, dlerror
   * 1000, (undocumented) принудительно запустить сборку мусора
   * 1007, (undocumented) set memory limit
   * 1009, (undocumented) get memory limit
   * 1008, (undocumented) get machine word size
   * 1022, (undocumented) set ticker
   * 1014, (undocumented) set slice
   * 1016, (undocumented) getenv
   * 1017, (undocumented) system (deprecated)
   * 1117, (undocumented) get memory stats

 * **62** (vm:version), возвращает версию виртуальной машины в виде точечной пары ("OL" . "1.1.0")

##### Команды встроенной в язык хеш-таблицы
В ol встроена поддержка быстрых однонаправленных хеш-таблиц, реализацию которых можно найти в библиотеке (owl ff). Эти таблицы представляют из себя красно-черное бинарное дерево, работа с которым построена в полном согласии с функциональной парадигмой программирования. Таким образом при добавлении в дерево новых элементов старое дерево не меняется. Однако не надо думать, что новая таблица является полным дублированием старой с соответствующими большими накладными расходами - это не так, дублируются только те элементы дерева, которые требуют перебалансировки и/или изменения. Желающие могут обратиться к [исходному коду](https://github.com/yuriy-chumak/ol/blob/master/owl/ff.scm).

 * **49** (ff-apply)
 * **43** (ff:red)
 * **42** (ff:black)
 * **46** (ff:toggle)
 * **41** (ff:red?)
 * **37** (ff:right?)

##### Сводная таблица команд виртуальной машины

|   #  | o0        | o1        | o2     | o3   | o4       | o5     | o6     | o7    |
|:-----|:---------:|:---------:|:------:|:----:|:--------:|:------:|:------:|:-----:|
|**0o**|    ---    |    REFI   |  GOTO  |OCL-C |OCL-P     | MOV2   |CL1-C   |CL1-P  |
|**1o**| JEQ       |    MOVE   |set-ref!|      |          | LDI    | LD     | type  |
|**2o**| JP        |ARITY-ERROR|        |      | apply    |        | cast   | NEW   |
|**3o**| RET       |    JAF    | DIV    | SYS  |endianness|wordsize|fxmax   |xmbits |
|**4o**|tuple-apply|           |        |unreel| size     |FFRIGHTQ| ADD    | MUL   |
|**5o**| SUB       | FFREDQ    |MKBLACK |MKRED | less?    |set-ref |FFTOGGLE| ref   |
|**6o**| raw?      | ff-apply  | RUN    | cons | car      |cdr     | EQ     | AND   |
|**7o**| LOR       |    XOR    | SHR    | SHL  | RAW      |clock   |version |syscall|

TBD.

# Lola-2: A Logic Description Language - Язык описания логики

```
N. Wirth, 24.4.1994 / 1.9.
```
Lola - это нотация (форма, язык) для описания цифровых схем (логики) - HDL язык описания архитектуры.
Во многом он напоминает язык программирования. Однако тексты `Lola` описывают статические схемы, а не динамические процессы.
Объектами, встречающимися в описании, могут быть переменные, представляющие сигналы или регистры. Их значения
определяются как выражения других объектов и операторов, представляющих logic gates (логические вентили).

## 0. Vocabulary

Словарь запас `Lola` состоит из специальных символов (или пар) и так называемых зарезервированных слов, которые
не могут быть выбраны в качестве идентификаторов.

Специальные символы:

```
~ & | ^ + - * = # < <= > >=
( ) [ ] { } ->. , ; : := '!
```
Зарезервированные слова:

```
BEGIN CONST END IN INOUT MODULE OUT REG TS TYPE VAR
```
## 1. Identifiers, integers, and comments - Идентификаторы, целые числа и комментарии

Идентификаторы используются для обозначения констант, переменных и типов.

```
identifier = letter {letter | digit}.
integer = digit {digit} ["H"].
```
Пример: Lola begin 100 100H 0FFH

Capital and lower case letters are considered as distinct. If an integer is followed by the capital letter
H it is in hexadecimal form with digits 0, 1, ..., 9, and A ... F.

Заглавные и строчные буквы рассматриваются как разные. Если за целым числом следует заглавная буква, то оно представлено в шестнадцатеричной форме с цифрами `0, 1, ..., 9` и `A..F`.

Комментарии представляют собой последовательности символов, заключенных в скобки (* и *), и они могут встречаться
между любыми двумя символами в тексте Lola.

## 2. Simple types and array types - простые типы и типы "массив"

Every variable in Lola has a type. The elementary type is BIT (BInary digiT), and its variables
assume the values 0 or 1.

Каждая переменная в Lola имеет тип. Элементарным типом является BIT (двоичная цифра), и такая переменная
принимает значения `0` или `1`.

```
type = {"[" expression "]"} SimpleType.
SimpleType = identifier |."MODULE" unit ";" identifier.
TypeDeclaration = identifier "=" type.
```
Тип массива задается формой `[n]T`, где `T` - тип элементов, а целое число `n`
указывает количество элементов. В выражениях отдельные элементы обозначаются индексом.
Индексы имеют значения от `0` до длины массива минус `1`.

Пример: [10] BIT [8][16] BIT

Идентификаторы `BIT`, `BYTE` и `WORD` объявлены заранее. Последние обозначают ширину, соответственно, в 8 и 32 бит.

## 3. Определение констант

Объявления констант служат для введения идентификаторов, обозначающих постоянное числовое значение.

```
ConstDeclaration = identifier "=" integer ";".
```

## 4. Variable and register declarations - объявление переменных и регистров

Variable declarations introduce registers and variables and associate a type with them. All variables
declared in an identifier list have the same type. Variables of type [n] BIT are called bitstrings.

Объявления переменных вводят регистры и переменные, так же связывают с ними тип. Все переменные
, объявленные в списке идентификаторов, имеют один и тот же тип. Переменные типа `[n] BIT` называются битовыми строками.

```
varlist = identifier {"," identifier} ":" type.
```
Пример: 
```
x, y: BIT     a: [32] BIT     R: BIT     Q (clk50): [4] BIT
```

If a register declaration contains an expression, this denotes the register's clock. The default is a
variable clk, which must be declared (see also Section 7).

Если объявление регистра содержит выражение, это обозначает тактовый сигнал регистра. По умолчанию используется
переменная clk, которая должна быть объявлена (см. также раздел 7).

## 5. Expressions - выражения

Выражения служат для объединения переменных с операторами для определения новых значений. Операторами являются
логическое отрицание, конъюнкция, дизъюнкция и разность или арифметическая сумма и arithmetic difference. Элементы
массива выбираются по индексу. (например, a.5, a[10]).

```
| logical disjunction (or)
^ logical difference (exclusive or)
& logical conjunction (and)
~ logical negation (not)
+ arithmetic sum
- arithmetic difference
```

Операндами являются регистры, переменные и константы. В то время как переменные обозначают значение, заданное
назначенным выражением, регистры обозначают значение выражения в предыдущем такте.
Регистры играют важную роль в последовательных схемах. Операнды двоичных операторов должны быть
одного типа.

```
expression = uncondExpr ["->" expression ":" expression].
uncondExpr = simpleExpr [ ("=" | "#" | "<" | "<=" | ">" | ">=") simpleExpr].
simpleExpr = ["+" | "-"] term {("|" | "^" | "+" | "-") term}.
term = factor {"&" factor}.
factor = variable | integer | "~" factor | constructor | "(" expression ")".
variable = identifier {selector}.
selector = "." factor | "[" expression [":" expression] "]".
constructor = "{" element {"," element} "}".
element = expression ["!" integer].
```

Определение `a[m : n]` обозначает диапазон индексированных элементов `a[m], ... , a[n]`.

Конструкторы обозначают битовые строки и представляют собой последовательности элементов. Длина (количество битов) каждого
элемента должна быть известна. Эта длина указывается в объявлении, а в данном случае констант
- явным целым числом. Например, `10'8` обозначает число 10, представленное 8 битами. Длина
битовой строки равна сумме длин ее элементов. Кроме того, за элементом может следовать
коэффициент репликации вида !n.

A conditional expression of the form z := cond -> x : y expresses represents a multiplexer. Cond
must be of type BIT. If cond yields 1, z is equal to x, otherwise to y.

Условное выражение вида `z := cond -> x : y` выражает мультиплексор. `Cond`
должно быть типа `BIT`. Если значение связи равно 1, то `z` будет равно `x`, иначе `y`.

Пример: 
```
x count 1 100 a.4 a[20] операнды
a[15 : 8] диапазон
{u, a.4, a[25:20], 0'8, 15'4} конкатенация (20 bit)
(x & y) | (z & w) (m + n) + 10 простое выражение
a.5 -> b : c выражение
```

## 6. Assignments and statements

Присвоения служат для определения значения регистра или переменной, которое задается выражением.
Присвоение должно пониматься как определение переменной (в отличие от определения идентификатора)


In an assignment v := x, v and x do not have the same roles, and this asymmetry is
emphasized by the use of the symbol := instead of the symmetric equal sign. v and x must be of the
same type.

В присвоении `v := x`, `v` и `x` имеют  разные роли, и эта асимметрия
подчеркивается использованием символа `:=` вместо симметричного знака равенства. `v` и `x` должны быть
одного типа.

```
assignment = variable ":=" expression..
```
Пример: 
```
x := y & z
a := {x, y, z}
R := rst -> 0 : enb -> x : R
```

Every variable and register can be assigned in only one single assignment statement, and the
assignment must be to the entire variable (not to elements). The only exception is the indexed
assignment to an array of registers (e.g. R[n] := x).

For example, instead of

Каждая переменная и регистр могут быть назначены только в одном операторе присваивания, и
присваивание должно быть всей переменной (не элементам). Единственным исключением является индексированное
присваивание массиву регистров (например, `R[n] := x`).

Например, вместо такого присваивания

```
a.0 := x; a.1 := x+y; a.2 := x-y; a.3 := x*y
```
должно быть использовано такое одиночное присваивание:

```
a := {x, x+y, x-y, x*y)
```

Операторы являются либо назначениями, либо экземплярами модулей (см. следующий раздел), либо TSgates.

```
statement = [assignment | instantiation | TSgate].
StatementSequence = statement {";" statement}.
```
Слово TSgate (TSgate расшифровывается как tri-state) используется для параметров порта типа INOUT.

```
TSgate = "TS" "(" iogate "," input "," output "," control ")".
iogate = variable.
input = variable. from interface
output = expression. to interface
control = expression. 1 for output, 0 for input from interface
```
Пример:
```
TS(io, in, out, ~wr)
TS(io[k], in[k], out[k], ctrl[k])
```

Параметрами могут быть простые переменные или массивы. Они должны быть одного типа, за исключением того, что
управляющий параметр может иметь тип BIT, даже если другие параметры являются массивами. В этом случае он
управляет всеми элементами массивов

## 7. Modules - модули

A module specifies constants, types, variables, registers, and assignments to variables and
registers. Modules are specified as types, and variables can be declared of this type. This implies
that modules can be replicated.

Модуль определяет константы, типы, переменные, регистры и присвоения переменным и
регистрам. Модули задаются как типы, и переменные могут быть объявлены с этим типом. Это подразумевает
, что модули могут реплицироваться.

```
ModuleType = "MODULE" ["*"] unit ";".
paramlist = ("IN" | "OUT" | "INOUT") varlist.
unit = "(" paramlist {";" paramlist} ")" ("^" | ";"
["CONST" {ConstDeclaration}]
["TYPE" {TypeDeclaration}]
{("VAR" | "REG" "(" expression ")") {varlist ";"}}
["BEGIN" StatementSequence] "END").
```
Выражение после символа REG определяет тактовый сигнал регистра.

Пример модуля - типа:

```
TYPE Counter = MODULE (IN clk, rst, enb: BIT; OUT data: WORD);
  REG (clk) R: Word;
BEGIN data := R;
  R := ~rst -> 0 : enb -> R + 1 : R
END Counter
```
Создание переменной такого типа `(cnt: Counter)`, может выглядеть так:

```
cnt (clock, reset, enable, val)
```
где `clock`, `reset`, `enable` и `val` - это переменные, называемые фактическими параметрами. Они должны быть выражениями
соответствующих типов.

```
instantiation = identifier selector "(" expression {"," expression} ")".
```
"Основная программа" имеет вид

```
module = "MODULE" identifier unit identifier ".".
```
Evidently, a "main program" is the combined declaration of an (anonymous) module type and of a
single instance. The identifier at the end of the module's declaration must be the same as the one
following the symbol MODULE.

Очевидно, что "основная программа" - это комбинированное объявление (анонимного) типа модуля в
единственном экземпляре. Идентификатор в конце объявления модуля должен совпадать с идентификатором,
следующим за символом MODULE.
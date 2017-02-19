# Функции высокого порядка
Функции могут быть возвращены как результат и передаваться как параметры, поэтому называются _функциями высокого порядка_.

```scala
def sumInts(a: Int, b: Int):
	if (a < b) 0 else a + sumInts(a + 1, b)
	
def cube(x: Int): Int: x * x * x

def sumCubes(a: Int, b: Int): Int = 
	if (a > b) 0 else cube(a) + sumCubes(a + 1, b)
	
def sumFactorials(a: Int, b: Int): Int =
	...
```

Видим общий паттерн в функциях. Как можем мы выразить этот паттерн в отдельной функции?

```scala
def sum(f: Int => Int, a: Int, b: Int): Int =
	if (a > b) 0
	else f(a) + sum(f, a + 1, b)
```

Используя соответствующие `f` мы реализовываем нужный функционал. Мы переиспользуем паттерн, который представили в функции `sum`. 

## Тип функции

Например, `Int => Int`. Проблема, мы все еще должны определять функции типа

```scala
def cube(x: Int): Int = ...
```

Хотим тратить на это меньше кода. Мы будем использовать для этого _анонимные функции_. Пример:

```scala
// example

(x: Int) => x * x * x

// another example

(x: Int, y: Int) => x + y
```

Анонимные функции являются синтаксическим сахаром, так как везде могут быть заменены обычным определением в блоке. 

Используя анонимные функции, мы получаем лаконичный код

```scala
def sumCubes(a: Int, b: Int) = sum(x => x * x * x, a, b)
```

## Каррирование (currying)

Однако, можем ли мы избавиться также от параметров `a` и `b`? 

```scala
def sum(f: Int => Int): (Int, Int) => Int = {
	def sumF(a: Int, b: Int): Int = 
		if (a > b) 0
		else f(a) + sumF(a + 1, b)
	sumF
}
```
Тогда можем переопределить `sumCubes`

```scala
def sumCubes = sum(cube)
```

В этом примере мы можем избежать определения `sumCubes` в принципе, если напишем так

```scala
sum(cube)(1, 10)
```
Здесь

* `sum(cubes)` применяет функцию `sum` к функции `cubes` и возвращает функцию _суммы кубов_
* `sum(cubes)` эквивалентна `sumCubes`
* Эта функция затем применяется к аргументам `(1, 10)`

Определение функций, которые возвращают функции очень фостребовано в функциональном программировании, поэтому обладает особым синтаксисом в Scala. последующее определение `sum` эквивалентно определению с вложенной функцией `sumF`, но короче

```scala
def sum(f: Int => Int)(a: Int, b: Int): Int =
	if (a > b) 0 else f(a) + sum(f)(a + 1, b)
```

В общем случае функции с несколькими списками параметров разворачиваются следующим образом:

```scala
def f(args[1])...(args[n]) = E
// equivalent ⇅
def f(args[1])...(args[n-1]) = {def g(args[n]) = E; g}
// or shorter ⇅
def f(args[1])...(args[n-1]) = (args[n] => E)
// ⇅
def f = (args[1] => (args[2] => (... (args[n] => E)...))
```

Этот стиль определения функций называется _каррированием_ (currying).

Тип функции `sum` записывается как `(Int => Int) => (Int, Int) => Int`.

_Пример: тотальное обобщение_

```scala
def mapReduce(f: Int => Int, combine: (Int, Int) => Int, zero: Int)(a: Int, b: Int) : Int =
	if (a > b) zero
	else combine(f(a), mapReduce(f, combine, zero)(a + 1, b)
```

## Обзор синтаксиса языка Scala

Опишем синтаксис языка Scala с помощью [расширенной формы Бэкуса-Наура](https://ru.wikipedia.org/wiki/Расширенная_форма_Бэкуса_—_Наура). 

* `|` ообозначает "или"
* `[...]` опциональный элемент (0 или 1)
* `{...}` повторение (0 или более)

```
Type 				= Simple Type | Function Type
Function Type 		= Simple Type '=>' Type | '(' [Types] ')' '=>' Type
Simple Type			= Ident
Types				= Type {',' Type}

Expr 				= InfixExpr | FunctionExpr |
 					  if '(' Expr ')' Expr else ExprInfixExpr 			= PrefixExpr | InfixExpr Operator InfixExprOperator 			= identPrefixExpr 			= [ '+' | '-' | '!' | '~' ] S i m p l e E x p rSimpleExpr 			= ident | literal | SimpleExpr '.' ident					  | BlockFunctionExpr 		= Bindings '=>' ExprBindings 			= ident [ ':' SimpleType ]					  | '(' [ Binding { ',' Binding }] ')'Binding 			= ident [ ': ' Type ]Block 				= '{' { Def ';'} Expr '}'
```

Тип (`Type`) может быть 

* Численным типом: `Int`, `Double` (`Byte`, `Short`, `Char`, `Long`, `Float`)
* Логическим типом: `Boolean`
* Строковым типом: `String`
* Функциональным типом: например `Int => Int` и прочее

Выражение может быть

* *Идентификатором*: `x`, `isGoodEnough`
* *Литералом*: `0`, `1.0`, `"abc"`
* *Применением функции*: `f(x)`
* *Применением оператора*: `-x, x + y`
* *Извлечением (selection)*: `math.abs`
* *Условным выражением*: `if (false) 1 else 0`
* *Блоком*
* *Анонимной функцией* 

...

# Объекты и классы

Будем строить класс рациональных чисел ($x/y$, где $x$ — делимое (numerator), $y$ — делитель (denominator)).

```scala
class Rational(x: Int, y: Int) {
	def numer = x
	def denom = y
}
```

Полсле исполнения этого кода появляются две новые сущности:

* Новый _тип_ `Rational`
* _Конструктор_ `Rational` для создания экземпляров этого класса

Эти сущности содержатся в разных неймспейсах (типов и значений), то есть они не портят друг друга.

```scala
val x = new Rational(1, 2)
x.denom
```

Функция сложения рациональных чисел

```scala
def addRational(r: Rational, s: Rational): Rational =	new Rational(		r.numer * s.denom + s.numer * r.denom,		r.denom * s.denom)
```

Мы хотим засунуть эту функцию внутрь класса, чтобы она стала методом.

```scala
class Rational(x: Int, y: Int) {
	def numer = x
	def denom = y
	
	def add(that: Rational): Rational =		new Rational(			numer * that.denom + that.numer * denom,			denom * that.denom)
			
	override def toString = numer + "/" + denom
	
	def neg: Rational = new Rational(-numer, denom)
	
	def sub(that: Rational) = add(that.neg)
}

new y = Rational(2, 3)
new z = Rational(2, 9)
x.add(y).sub(z)
```

Хотим, чтобы не было, например, `49/70`, так как число не до конца упрощено. Вместо того, чтобы вызывать "упрощатель" каждый раз в конце вычисления той или иной операции, мы будем использовать его при конструкции объекта, что более логично, так как объекты являются неизменяемыми. Улучшаем класс

```scala
class Rational(x: Int, y: Int) {
	private def gcd(a: Int, b: Int): Int =>
		if (b == 0) a else gcd(b, a % b)
		
	private def g = gcd(x, y)
	def numer = x / g // computed every time. We can change to val
	def denom = y / g
	
	def add(that: Rational): Rational =		new Rational(			numer * that.denom + that.numer * denom,			denom * that.denom)
			
	override def toString = numer + "/" + denom
	
	def neg: Rational = new Rational(-numer, denom)
	
	def sub(that: Rational) = add(that.neg)

}

new y = Rational(2, 3)
new z = Rational(2, 9)
x.add(y).sub(z)
```

То, что мы можем менять имплементацию класса без смены поведения его объектов называется _абстракцией данных (data abstraction)_.

Добавляем еще пару методов

```scala
def less(that: Rational) = numer * that.denom < that.numer * denom

def max(that: Rational) = if (this.less(that)) that else this
```

Хотим защититься от того, что `denom` может быть равен нулю. Для этого засовываем в определение класса `require`

```scala
	class Rational(x: Int, y: Int) {
		require(y != 0, "Denominator must be non-zero")
		...
	}
```

`require` берет условие и строку, если условие нарушается, то появляется исключение, содержащее строку. Вместо него могли использовать `assert(x >= 0)`, однако он используется для других целей (проверка корректности кода), тогда как `require` проверяет пришедшие аргументы.

Тело определения класса является _главным конструктором класса (primary constructor)_. Если мы хотим перегрузить конструктор, то мы должны явно определить перегрузку внутри класса, которая будет использовать основной конструктор, например в случае класса `Rational`

```scala
	...
	def this(x: Int): this(x, 1)
	...
new Rational(2)
```

## Вычисление

`new Clazz(n_1...n_m)`вычисляется как Call-by-Value функция, после этого объект уже является _значением_.

`class Clazz(x_1...x_m) {... def f(y_1...y_n) = B ...}` 

Тогда выражение `new Clazz(v_1...v_m).f(w_1...w_n)` вычисляется следующим образом (квадратные скобки обозначают замену)

```
[w_1/y_1...w_n/y_n][v_1/x_1...v_m/x_m][new Clazz(v_1...v_m)/this]B
```	

## Операторы

Каждый метод с параметром может быть использован как инфиксный оператор. Например, для метода `x.add(y)` можно записать `x add y`, эти конструкции эквивалентны.

В Scala операторы могут быть

* **(Alphanumeric)**: состоять из алфавитных симоволов, `_` и цифр
* **(Symbolic)**: начинаться с операторного символа, за которым следуют другие операторные символы
* Идентификаторы первого типа могут заканчиваться постфиксом `_` + операторные символы

Таким образом эти последовательности символов являются идентификаторами в Scala: `x1, *, +?%&, vector_++, counter_=`.

Таким образом, мы можем переопределить метод `less` в определении класса

```scala
	def < (that: Rational) = numer * that.denom < that.numer * denom
```
и так далее

**Важно**: для определения унарного `-` нужно использовать имя `unary_-`. Это является специальным соглашением в языке Scala.

### Приоритеты операторов

По первому символу оператора: 
```
(all letters)|^&< >= !:+ -* / %(all other special characters)
```

```
((a + b) ^? (c ?^ d)) less ((a ==> b) | c)
```
---
layout: page
title: Функции
---

# Функции

## Содержание
- [Функция в математике](#функция-в-математике)
- [Чистые функции](#чистые-функции)
- [Преимущества чистых функций](#преимущества-чистых-функций)
- [Impure функции и край мира](#impure-функции-и-край-мира)
- [Рекурсия](#рекурсия)
- [Параметры функций](#параметры-функций)
- [Функции как значения](#функции-как-значения)
- [Композиция функций](#композиция-функций)
- [Частичные функции (Partial Functions)](#частичные-функции-partial-functions)

---

## Функция в математике

В математике **функция** — это строгое отображение между двумя множествами, где каждому элементу из области определения соответствует ровно один элемент из области значений.

Формально функция `f: A → B` — это отношение, которое каждому элементу `a ∈ A` ставит в соответствие единственный элемент `b ∈ B`.

```
f(x) = y, где x ∈ A, y ∈ B
```

### Ключевые свойства математической функции:

1. **Детерминированность** — для одного и того же входного значения всегда возвращается один и тот же результат
2. **Тотальность** — функция определена для всех элементов области определения
3. **Отсутствие побочных эффектов** — функция только вычисляет результат, не изменяя ничего во внешнем мире

```scala
// Пример математической функции: квадрат числа
// f(x) = x²
def square(x: Int): Int = x * x

square(3) // всегда 9
square(3) // снова 9, и так будет всегда
```

---

## Чистые функции

**Чистая функция (Pure Function)** — это функция в программировании, которая полностью соответствует математическому определению функции.

### Два главных свойства чистой функции:

#### 1. Детерминированность (Referential Transparency)

Для одних и тех же входных данных функция всегда возвращает один и тот же результат.

```scala
// Чистая функция — детерминированная
def add(a: Int, b: Int): Int = a + b

add(2, 3) // всегда 5
add(2, 3) // всегда 5

// Нечистая функция — недетерминированная
def randomAdd(a: Int): Int = a + scala.util.Random.nextInt(10)

randomAdd(5) // может быть 7
randomAdd(5) // может быть 12
```

#### 2. Отсутствие побочных эффектов (No Side Effects)

Функция не изменяет состояние внешнего мира: не модифицирует глобальные переменные, не пишет в файлы, не отправляет сетевые запросы, не выводит данные на экран.

```scala
// Чистая функция — без побочных эффектов
def multiply(a: Int, b: Int): Int = a * b

// Нечистая функция — есть побочный эффект (вывод в консоль)
def multiplyWithLog(a: Int, b: Int): Int = {
  println(s"Multiplying $a by $b")  // побочный эффект!
  a * b
}

// Нечистая функция — изменяет внешнее состояние
var counter = 0
def incrementAndAdd(a: Int): Int = {
  counter += 1  // побочный эффект!
  a + counter
}
```

### Примеры побочных эффектов:

| Побочный эффект | Пример |
|-----------------|--------|
| Изменение переменной | `counter += 1` |
| Запись в файл | `File.write(...)` |
| Чтение из файла | `File.read(...)` |
| Сетевой запрос | `Http.get(url)` |
| Вывод в консоль | `println(...)` |
| Чтение текущего времени | `System.currentTimeMillis()` |
| Генерация случайных чисел | `Random.nextInt()` |
| Изменение базы данных | `db.insert(...)` |

---

## Преимущества чистых функций

### 1. Предсказуемость поведения

Чистые функции легко понять и отладить, потому что их поведение полностью определяется входными параметрами.

```scala
// Легко понять, что делает эта функция
def calculateDiscount(price: Double, discountPercent: Double): Double =
  price * (1 - discountPercent / 100)

// Можно заменить вызов функции её результатом (referential transparency)
val price1 = calculateDiscount(100, 20)  // 80.0
val price2 = 80.0                         // эквивалентно

// Это позволяет рассуждать о коде как о математических уравнениях
```

### 2. Простота рефакторинга

Чистые функции можно безопасно перемещать, переименовывать и изменять, не беспокоясь о скрытых зависимостях.

```scala
// Можно безопасно извлечь в отдельный модуль
object PriceCalculator {
  def calculateDiscount(price: Double, discountPercent: Double): Double =
    price * (1 - discountPercent / 100)

  def applyTax(price: Double, taxRate: Double): Double =
    price * (1 + taxRate / 100)

  // Легко комбинировать чистые функции
  def finalPrice(price: Double, discount: Double, tax: Double): Double =
    applyTax(calculateDiscount(price, discount), tax)
}
```

### 3. Простота тестирования

Чистые функции тестируются тривиально — не нужно настраивать окружение, моки или состояние.

```scala
// Тест чистой функции — просто и понятно
class CalculatorSpec extends AnyFlatSpec {
  "calculateDiscount" should "apply percentage discount" in {
    assert(calculateDiscount(100, 20) == 80.0)
    assert(calculateDiscount(50, 10) == 45.0)
    assert(calculateDiscount(200, 0) == 200.0)
  }
}

// Сравните с тестированием функции с побочными эффектами:
// - нужно мокать базу данных
// - нужно настраивать файловую систему
// - нужно изолировать состояние между тестами
```

### 4. Безопасная параллелизация

Чистые функции можно безопасно выполнять параллельно, так как они не имеют разделяемого состояния.

```scala
val numbers = (1 to 1000000).toList

// Безопасно распараллеливается, потому что square — чистая функция
val squares = numbers.par.map(square)
```

---

## Impure функции и край мира

Несмотря на все преимущества чистых функций, **impure функции необходимы** — программа должна как-то взаимодействовать с внешним миром: читать ввод пользователя, выводить результаты, сохранять данные.

### Концепция "края мира" (Edge of the World)

В функциональном программировании принято **изолировать** побочные эффекты на границах приложения:

```
┌─────────────────────────────────────────────────────┐
│                   Край мира (Impure)                │
│  ┌─────────────┐                  ┌──────────────┐  │
│  │ Input Layer │                  │ Output Layer │  │
│  │ - HTTP      │                  │ - Database   │  │
│  │ - Files     │                  │ - Console    │  │
│  │ - Console   │                  │ - Files      │  │
│  └──────┬──────┘                  └──────▲───────┘  │
│         │                                │          │
│         ▼                                │          │
│  ┌──────────────────────────────────────────────┐   │
│  │            Чистое ядро (Pure Core)           │   │
│  │                                              │   │
│  │   Бизнес-логика, вычисления, трансформации  │   │
│  │                                              │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### Пример архитектуры с изоляцией эффектов

```scala
// ===== Чистое ядро (Pure Core) =====
case class User(id: Long, name: String, email: String)
case class Order(id: Long, userId: Long, amount: Double)

// Чистые функции бизнес-логики
object OrderLogic {
  def calculateTotal(orders: List[Order]): Double =
    orders.map(_.amount).sum

  def applyDiscount(total: Double, userLevel: Int): Double =
    total * (1 - userLevel * 0.05)

  def formatReceipt(user: User, total: Double): String =
    s"Receipt for ${user.name}: $$${total}"
}

// ===== Край мира (Impure Shell) =====
object Main extends App {
  // Input layer (impure)
  val userId = scala.io.StdIn.readLine("Enter user ID: ").toLong

  // В реальности здесь был бы запрос к БД
  val user = User(userId, "John", "john@example.com")
  val orders = List(Order(1, userId, 100), Order(2, userId, 50))

  // Pure core — вся логика чистая
  val total = OrderLogic.calculateTotal(orders)
  val discounted = OrderLogic.applyDiscount(total, userLevel = 2)
  val receipt = OrderLogic.formatReceipt(user, discounted)

  // Output layer (impure)
  println(receipt)
}
```

### Почему это важно?

- **Тестируемость**: чистое ядро легко тестировать без моков
- **Понятность**: побочные эффекты сосредоточены в одном месте
- **Поддерживаемость**: изменения в логике не затрагивают I/O и наоборот

---

## Рекурсия

**Рекурсия** — это способ определения функции через саму себя. В функциональном программировании рекурсия является основным инструментом для организации повторяющихся вычислений вместо императивных циклов.

### Простой пример рекурсии

```scala
// Вычисление факториала рекурсивно
def factorial(n: Int): Int =
  if (n <= 1) 1
  else n * factorial(n - 1)

factorial(5)  // 5 * 4 * 3 * 2 * 1 = 120
```

Как это работает:
```
factorial(5)
= 5 * factorial(4)
= 5 * (4 * factorial(3))
= 5 * (4 * (3 * factorial(2)))
= 5 * (4 * (3 * (2 * factorial(1))))
= 5 * (4 * (3 * (2 * 1)))
= 120
```

### Проблема: переполнение стека

Каждый рекурсивный вызов добавляет новый фрейм в стек вызовов. При глубокой рекурсии это приводит к `StackOverflowError`:

```scala
factorial(100000)  // StackOverflowError!
```

### Хвостовая рекурсия (Tail Recursion)

**Хвостовая рекурсия** — это особая форма рекурсии, при которой рекурсивный вызов является **последней операцией** в функции.

```scala
// Обычная рекурсия — НЕ хвостовая
def factorial(n: Int): Int =
  if (n <= 1) 1
  else n * factorial(n - 1)  // после вызова ещё умножение!

// Хвостовая рекурсия
def factorialTailRec(n: Int): Int = {
  @tailrec
  def loop(current: Int, accumulator: Int): Int =
    if (current <= 1) accumulator
    else loop(current - 1, current * accumulator)  // вызов — последняя операция

  loop(n, 1)
}
```

### Почему хвостовая рекурсия лучше?

Компилятор Scala оптимизирует хвостовую рекурсию в обычный цикл (**tail call optimization**), что позволяет избежать роста стека:

```scala
import scala.annotation.tailrec

@tailrec  // компилятор проверит, что рекурсия действительно хвостовая
def factorialTailRec(n: Int, acc: Int = 1): Int =
  if (n <= 1) acc
  else factorialTailRec(n - 1, n * acc)

factorialTailRec(100000)  // Работает! Нет переполнения стека
```

### Примеры хвостовой рекурсии

```scala
import scala.annotation.tailrec

// Сумма списка
@tailrec
def sum(list: List[Int], acc: Int = 0): Int = list match {
  case Nil => acc
  case head :: tail => sum(tail, acc + head)
}

// Длина списка
@tailrec
def length[A](list: List[A], acc: Int = 0): Int = list match {
  case Nil => acc
  case _ :: tail => length(tail, acc + 1)
}

// Разворот списка
@tailrec
def reverse[A](list: List[A], acc: List[A] = Nil): List[A] = list match {
  case Nil => acc
  case head :: tail => reverse(tail, head :: acc)
}

// Числа Фибоначчи
@tailrec
def fibonacci(n: Int, a: BigInt = 0, b: BigInt = 1): BigInt =
  if (n == 0) a
  else fibonacci(n - 1, b, a + b)
```

### Паттерн аккумулятора

Преобразование обычной рекурсии в хвостовую часто требует добавления **аккумулятора** — параметра, который накапливает результат:

```scala
// Без аккумулятора (не хвостовая)
def power(base: Int, exp: Int): Int =
  if (exp == 0) 1
  else base * power(base, exp - 1)

// С аккумулятором (хвостовая)
@tailrec
def powerTailRec(base: Int, exp: Int, acc: Int = 1): Int =
  if (exp == 0) acc
  else powerTailRec(base, exp - 1, acc * base)
```

---

## Параметры функций

Scala предоставляет богатые возможности для работы с параметрами функций.

### Параметры по умолчанию (Default Parameters)

```scala
def greet(name: String, greeting: String = "Hello"): String =
  s"$greeting, $name!"

greet("Alice")           // "Hello, Alice!"
greet("Bob", "Hi")       // "Hi, Bob!"
greet("Charlie", "Hey")  // "Hey, Charlie!"
```

### Именованные параметры (Named Parameters)

Позволяют передавать аргументы в любом порядке:

```scala
def createUser(name: String, age: Int, email: String): User =
  User(name, age, email)

// Позиционные аргументы
createUser("Alice", 25, "alice@example.com")

// Именованные аргументы — порядок не важен
createUser(
  email = "bob@example.com",
  name = "Bob",
  age = 30
)

// Комбинирование: позиционные должны идти первыми
createUser("Charlie", email = "charlie@example.com", age = 28)
```

### Передача по имени (By-Name Parameters)

Параметры by-name вычисляются **лениво** — только в момент использования:

```scala
// By-value: аргумент вычисляется сразу при вызове
def byValue(x: Int): Unit = {
  println("byValue called")
  println(x)
  println(x)
}

// By-name: аргумент вычисляется каждый раз при обращении
def byName(x: => Int): Unit = {
  println("byName called")
  println(x)  // вычисляется здесь
  println(x)  // вычисляется снова
}

// Демонстрация разницы
var counter = 0
def increment(): Int = { counter += 1; counter }

byValue(increment())  // выведет: 1, 1 (вычислено один раз)
counter = 0
byName(increment())   // выведет: 1, 2 (вычислено дважды)
```

Практическое применение by-name параметров:

```scala
// Условное выполнение — логируем только если нужно
def log(level: String)(message: => String): Unit =
  if (level == "DEBUG") println(s"[$level] $message")

// Дорогая операция не выполнится, если level != "DEBUG"
log("INFO")(expensiveComputation())  // expensiveComputation не вызовется

// Реализация собственных управляющих конструкций
def unless(condition: Boolean)(block: => Unit): Unit =
  if (!condition) block

unless(list.isEmpty) {
  println("List is not empty")
  processItems(list)
}
```

### Переменное число аргументов (Varargs)

```scala
def sum(numbers: Int*): Int = numbers.sum

sum()           // 0
sum(1)          // 1
sum(1, 2, 3)    // 6
sum(1, 2, 3, 4) // 10

// Передача коллекции как varargs с помощью _*
val nums = List(1, 2, 3, 4, 5)
sum(nums*)      // Scala 3
sum(nums: _*)   // Scala 2
```

### Типы как параметры (Type Parameters / Generics)

```scala
// Обобщённая функция — работает с любым типом
def identity[A](x: A): A = x

identity(42)        // Int: 42
identity("hello")   // String: "hello"
identity(List(1,2)) // List[Int]: List(1, 2)

// Несколько типовых параметров
def pair[A, B](a: A, b: B): (A, B) = (a, b)

pair(1, "one")      // (Int, String): (1, "one")
pair("two", 2.0)    // (String, Double): ("two", 2.0)

// Ограничения на типы (Type Bounds)
def max[A <: Ordered[A]](a: A, b: A): A =
  if (a > b) a else b

// Контекстные ограничения (Context Bounds)
def sorted[A: Ordering](list: List[A]): List[A] =
  list.sorted
```

---

## Функции как значения

### First-Class Citizen Functions

В функциональном программировании функции являются **значениями первого класса (first-class citizens)**. Это означает, что функции можно:

1. **Присваивать переменным**
2. **Передавать как аргументы** другим функциям
3. **Возвращать из функций**
4. **Хранить в структурах данных**

```scala
// 1. Присвоение переменной
val double: Int => Int = (x: Int) => x * 2

// 2. Передача как аргумент
val numbers = List(1, 2, 3, 4, 5)
numbers.map(double)  // List(2, 4, 6, 8, 10)

// 3. Возврат из функции
def multiplier(factor: Int): Int => Int =
  (x: Int) => x * factor

val triple = multiplier(3)
triple(5)  // 15

// 4. Хранение в структурах данных
val operations: Map[String, (Int, Int) => Int] = Map(
  "add" -> ((a, b) => a + b),
  "sub" -> ((a, b) => a - b),
  "mul" -> ((a, b) => a * b)
)

operations("add")(10, 5)  // 15
operations("mul")(10, 5)  // 50
```

### Реализация функций в Scala

В Scala функции — это объекты. Функция с N параметрами является экземпляром трейта `FunctionN`:

```scala
// Трейты функций в стандартной библиотеке
trait Function0[+R] { def apply(): R }
trait Function1[-T1, +R] { def apply(v1: T1): R }
trait Function2[-T1, -T2, +R] { def apply(v1: T1, v2: T2): R }
// ... до Function22

// Когда мы пишем:
val add: (Int, Int) => Int = (a, b) => a + b

// Scala создаёт объект:
val add: Function2[Int, Int, Int] = new Function2[Int, Int, Int] {
  def apply(a: Int, b: Int): Int = a + b
}

// Вызов функции — это вызов метода apply
add(2, 3)       // синтаксический сахар для:
add.apply(2, 3) // 5
```

### Анонимные функции (Lambda)

Анонимные функции — это функции без имени, определяемые на месте:

```scala
// Полный синтаксис
val add = (a: Int, b: Int) => a + b

// Сокращённый синтаксис с выводом типов
val numbers = List(1, 2, 3, 4, 5)

numbers.map((x: Int) => x * 2)  // полная форма
numbers.map(x => x * 2)          // тип выводится
numbers.map(_ * 2)               // placeholder синтаксис

// Несколько placeholder'ов
numbers.reduce(_ + _)            // (a, b) => a + b
numbers.reduce(_ * _)            // (a, b) => a * b

// Многострочные лямбды
numbers.map { x =>
  val doubled = x * 2
  val squared = doubled * doubled
  squared
}
```

### Функции vs Методы

В Scala есть различие между **функциями** и **методами**:

```scala
// Метод — определяется с помощью def, привязан к объекту/классу
def methodAdd(a: Int, b: Int): Int = a + b

// Функция — значение типа FunctionN
val functionAdd: (Int, Int) => Int = (a, b) => a + b
```

#### Ключевые различия:

| Характеристика | Метод | Функция |
|----------------|-------|---------|
| Определение | `def name(...)` | `val name = (...) => ...` |
| Является значением | Нет | Да |
| Может быть дженериком | Да | Нет (только при определении) |
| Поддержка varargs | Да | Нет |
| Параметры по умолчанию | Да | Нет |
| Именованные параметры | Да | Нет |

```scala
// Методы могут быть дженериками
def identity[A](x: A): A = x

// Функции фиксируют тип при создании
val identityInt: Int => Int = x => x
val identityString: String => String = x => x

// Методы поддерживают дефолтные параметры
def greet(name: String, greeting: String = "Hello"): String = s"$greeting, $name"

// Преобразование метода в функцию (eta-expansion)
val greetFn: (String, String) => String = greet
// или
val greetFn = greet _  // Scala 2
val greetFn = greet    // Scala 3 (автоматически)
```

---

## Композиция функций

**Композиция функций** — это создание новой функции путём последовательного применения нескольких функций. Если `f: A => B` и `g: B => C`, то их композиция `g ∘ f: A => C`.

### Операторы compose и andThen

```scala
val addOne: Int => Int = _ + 1
val double: Int => Int = _ * 2
val square: Int => Int = x => x * x

// compose: применяет функции справа налево (как в математике)
// (f compose g)(x) = f(g(x))
val doubleThenAddOne = addOne compose double
doubleThenAddOne(5)  // double(5) = 10, addOne(10) = 11

// andThen: применяет функции слева направо
// (f andThen g)(x) = g(f(x))
val addOneThenDouble = addOne andThen double
addOneThenDouble(5)  // addOne(5) = 6, double(6) = 12

// Цепочка композиций
val pipeline = addOne andThen double andThen square
pipeline(3)  // addOne(3)=4, double(4)=8, square(8)=64
```

### Практический пример композиции

```scala
case class User(name: String, email: String)

// Цепочка валидаций
val trimName: User => User = u => u.copy(name = u.name.trim)
val lowercaseEmail: User => User = u => u.copy(email = u.email.toLowerCase)
val validateName: User => Either[String, User] = u =>
  if (u.name.nonEmpty) Right(u) else Left("Name cannot be empty")

// Композиция трансформаций
val normalizeUser = trimName andThen lowercaseEmail

val user = User("  Alice  ", "ALICE@Example.COM")
normalizeUser(user)  // User("Alice", "alice@example.com")
```

### Каррирование (Currying)

**Каррирование** — это преобразование функции с несколькими аргументами в последовательность функций, каждая из которых принимает один аргумент.

```
f(a, b, c) → f(a)(b)(c)
```

```scala
// Некаррированная функция
def add(a: Int, b: Int): Int = a + b

// Каррированная функция
def addCurried(a: Int)(b: Int): Int = a + b

// Использование
add(2, 3)        // 5
addCurried(2)(3) // 5

// Частичное применение каррированной функции
val addTwo: Int => Int = addCurried(2)
addTwo(3)  // 5
addTwo(10) // 12
```

### Преобразование между формами

```scala
// Некаррированная функция
val add: (Int, Int) => Int = (a, b) => a + b

// Преобразование в каррированную форму
val addCurried: Int => Int => Int = add.curried

addCurried(2)(3)  // 5

// Обратное преобразование
val addUncurried: (Int, Int) => Int = Function.uncurried(addCurried)

// Каррирование методов
def multiply(a: Int, b: Int, c: Int): Int = a * b * c
val multiplyCurried = (multiply _).curried  // Int => Int => Int => Int

multiplyCurried(2)(3)(4)  // 24
```

### Частичное применение (Partial Application)

**Частичное применение** — это фиксация части аргументов функции с получением новой функции с меньшим числом аргументов.

```scala
def greet(greeting: String, name: String): String = s"$greeting, $name!"

// Частичное применение — фиксируем первый аргумент
val sayHello: String => String = greet("Hello", _)
val sayHi: String => String = greet("Hi", _)

sayHello("Alice")  // "Hello, Alice!"
sayHi("Bob")       // "Hi, Bob!"

// Частичное применение с несколькими параметрами
def sendEmail(from: String, to: String, subject: String, body: String): Unit =
  println(s"From: $from, To: $to, Subject: $subject\n$body")

// Фиксируем отправителя
val sendFromAdmin: (String, String, String) => Unit =
  sendEmail("admin@example.com", _, _, _)

sendFromAdmin("user@example.com", "Welcome", "Hello!")
```

### Практическое применение каррирования

```scala
// 1. Конфигурирование функций
def logger(level: String)(message: String): Unit =
  println(s"[$level] $message")

val info = logger("INFO")
val error = logger("ERROR")

info("Application started")   // [INFO] Application started
error("Connection failed")    // [ERROR] Connection failed

// 2. Создание DSL
def withTransaction[A](db: Database)(block: Connection => A): A = {
  val conn = db.getConnection()
  try {
    conn.beginTransaction()
    val result = block(conn)
    conn.commit()
    result
  } catch {
    case e: Exception =>
      conn.rollback()
      throw e
  } finally {
    conn.close()
  }
}

// Использование
val withMyDb = withTransaction(myDatabase)
withMyDb { conn =>
  conn.execute("INSERT INTO users ...")
}

// 3. Внедрение зависимостей
def fetchUser(httpClient: HttpClient)(userId: Long): User =
  httpClient.get(s"/users/$userId").as[User]

val fetchUserWithClient = fetchUser(productionClient)
val user = fetchUserWithClient(123)
```

---

## Частичные функции (Partial Functions)

**Частичная функция (Partial Function)** — это функция, которая определена не для всех значений входного типа, а только для некоторого подмножества.

В Scala частичные функции представлены трейтом `PartialFunction[A, B]`.

### Определение частичных функций

```scala
// Частичная функция через case
val divide: PartialFunction[Int, Int] = {
  case x if x != 0 => 100 / x
}

// Эта функция не определена для 0
divide(10)    // 10
divide(5)     // 20
divide(0)     // MatchError! Функция не определена для 0

// Проверка, определена ли функция для значения
divide.isDefinedAt(10)  // true
divide.isDefinedAt(0)   // false
```

### Примеры частичных функций

```scala
// Обработка только чётных чисел
val doubleEvens: PartialFunction[Int, Int] = {
  case x if x % 2 == 0 => x * 2
}

// Обработка определённых типов сообщений
sealed trait Message
case class TextMessage(text: String) extends Message
case class ImageMessage(url: String) extends Message
case class VideoMessage(url: String, duration: Int) extends Message

val handleText: PartialFunction[Message, String] = {
  case TextMessage(text) => s"Received text: $text"
}

val handleMedia: PartialFunction[Message, String] = {
  case ImageMessage(url) => s"Received image: $url"
  case VideoMessage(url, _) => s"Received video: $url"
}
```

### Комбинирование частичных функций

```scala
val handleText: PartialFunction[Message, String] = {
  case TextMessage(text) => s"Text: $text"
}

val handleImage: PartialFunction[Message, String] = {
  case ImageMessage(url) => s"Image: $url"
}

val handleVideo: PartialFunction[Message, String] = {
  case VideoMessage(url, duration) => s"Video: $url ($duration sec)"
}

// orElse — объединение частичных функций
val handleAll = handleText orElse handleImage orElse handleVideo

handleAll(TextMessage("Hello"))        // "Text: Hello"
handleAll(ImageMessage("cat.jpg"))     // "Image: cat.jpg"
handleAll(VideoMessage("movie.mp4", 120)) // "Video: movie.mp4 (120 sec)"
```

### Методы PartialFunction

```scala
val sqrt: PartialFunction[Double, Double] = {
  case x if x >= 0 => Math.sqrt(x)
}

// isDefinedAt — проверка определённости
sqrt.isDefinedAt(4.0)   // true
sqrt.isDefinedAt(-1.0)  // false

// lift — преобразование в обычную функцию, возвращающую Option
val safeSqrt: Double => Option[Double] = sqrt.lift
safeSqrt(4.0)   // Some(2.0)
safeSqrt(-1.0)  // None

// applyOrElse — применение с fallback
sqrt.applyOrElse(-1.0, (x: Double) => 0.0)  // 0.0

// andThen — композиция с обычной функцией
val sqrtThenDouble = sqrt andThen (_ * 2)
sqrtThenDouble(4.0)  // 4.0
```

### Использование с коллекциями

```scala
val numbers = List(-2, -1, 0, 1, 2, 3, 4)

// collect — применяет частичную функцию только к подходящим элементам
val positiveSquares = numbers.collect {
  case x if x > 0 => x * x
}
// List(1, 4, 9, 16)

// Эквивалентно filter + map, но более выразительно
val same = numbers.filter(_ > 0).map(x => x * x)

// Практический пример: парсинг значений
val inputs = List("1", "two", "3", "four", "5")

val validNumbers = inputs.collect {
  case s if s.forall(_.isDigit) => s.toInt
}
// List(1, 3, 5)
```

### Частичные функции vs Partial Application

Важно не путать эти два понятия:

| Частичная функция (Partial Function) | Частичное применение (Partial Application) |
|--------------------------------------|-------------------------------------------|
| Функция определена не для всех входов | Часть аргументов функции зафиксирована |
| `PartialFunction[A, B]` | Обычная функция с меньшей арностью |
| `{ case x if p(x) => ... }` | `f(fixedArg, _)` |

```scala
// Partial Function — не определена для всех Int
val divideBy: PartialFunction[Int, Int] = {
  case x if x != 0 => 100 / x
}

// Partial Application — зафиксирован делитель
def divide(a: Int, b: Int): Int = a / b
val divideHundredBy: Int => Int = divide(100, _)
```

---

## Заключение

В этой лекции мы рассмотрели:

- **Чистые функции** — основа функционального программирования, обеспечивающая предсказуемость и тестируемость
- **Рекурсию** и её оптимизированную форму — хвостовую рекурсию
- **Разнообразные параметры функций** в Scala: дефолтные, именованные, by-name, varargs и типовые параметры
- **Функции как значения первого класса**, их реализацию в Scala и отличие от методов
- **Композицию функций** через `compose` и `andThen`, а также каррирование и частичное применение
- **Частичные функции** — функции, определённые только на части области определения

Понимание этих концепций позволяет писать более выразительный, безопасный и легко тестируемый код в функциональном стиле.

---

[← Назад к содержанию](../index.html)

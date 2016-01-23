% Шаблоны сопоставления `match`

Шаблоны достаточно часто используются в Rust. Мы уже использовали их в разделе
[Связывание переменных][bindings], в разделе [Конструкция `match`][match], а
также в некоторых других местах. Давайте коротко пробежимся по всем
возможностям, которые можно реализовать с помощью шаблонов!

[bindings]: variable-bindings.html
[match]: match.html

Быстро освежим в памяти: сопоставлять с шаблоном литералы можно либо напрямую,
либо с использованием символа `_`, который означает *любой* случай:

```rust
let x = 1;

match x {
    1 => println!("один"),
    2 => println!("два"),
    3 => println!("три"),
    _ => println!("что угодно"),
}
```

Этот код напечатает `один`.

# Сопоставление с несколькими шаблонами

Вы можете сопоставлять с несколькими шаблонами, используя `|`:

```rust
let x = 1;

match x {
    1 | 2 => println!("один или два"),
    3 => println!("три"),
    _ => println!("что угодно"),
}
```

Этот код напечатает `один или два`.

# Деструктуризация

Если вы работаете с составным типом данных, вроде [`struct`][struct], вы можете
разобрать его на части («деструктурировать») внутри шаблона:

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { x, y } => println!("({},{})", x, y),
}
```

[struct]: structs.html

Мы можем использовать `:`, чтобы привязать значение к новому имени.

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { x: x1, y: y1 } => println!("({},{})", x1, y1),
}
```

Если нас интересуют только некоторые значения, мы можем не давать имена всем
составляющим:

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { x, .. } => println!("x равен {}", x),
}
```

Этот код напечатает `x равен 0`.

Вы можете использовать это в любом сопоставлении: не обязательно игнорировать
именно первый элемент:

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { y, .. } => println!("y равен {}", y),
}
```

Этот код напечатает `y равен 0`.

Можно произвести деструктуризацию любого составного типа данных — например,
[кортежей][tuples] и [перечислений][enums].

[tuples]: primitive-types.html#tuples
[enums]: enums.html

# Игнорирование связывания

Вы можете использовать в шаблоне `_`, чтобы проигнорировать соответствующее
значение. Например, вот сопоставление `Result<T, E>`:

```rust
# let some_value: Result<i32, &'static str> = Err("Здесь была какая-то ошибка");
match some_value {
    Ok(value) => println!("получили значение: {}", value),
    Err(_) => println!("произошла ошибка"),
}
```

В первой ветви мы привязываем значение варианта `Ok` к имени `value`. А в ветви
обработки варианта `Err` мы используем `_`, чтобы проигнорировать конкретную
ошибку, и просто печатаем общее сообщение.

`_` допустим в любом шаблоне, который связывает имена. Это можно использовать,
чтобы проигнорировать части большой структуры:

```rust
fn coordinate() -> (i32, i32, i32) {
    // создаём и возвращаем какой-то кортеж из трёх элементов
# (1, 2, 3)
}

let (x, _, z) = coordinate();
```

Здесь мы связываем первый и последний элемент кортежа с именами `x` и `z`
соответственно, а второй элемент игнорируем.

Похожим образом, в шаблоне можно использовать `..`, чтобы проигнорировать
несколько значений.

```rust
enum OptionalTuple {
    Value(i32, i32, i32),
    Missing,
}

let x = OptionalTuple::Value(5, -2, 3);

match x {
    OptionalTuple::Value(..) => println!("Получили кортеж!"),
    OptionalTuple::Missing => println!("Вот неудача."),
}
```

Этот код печатает `Получили кортеж!`.

# ref и ref mut

Если вы хотите получить [ссылку][ref], то используйте ключевое слово `ref`:

```rust
let x = 5;

match x {
    ref r => println!("Получили ссылку на {}", r),
}
```

Этот код напечатает `Получили ссылку на 5`.

[ref]: references-and-borrowing.html

Здесь `r` внутри `match` имеет тип `&i32`. Другими словами, ключевое слово `ref`
_создает_ ссылку, для использования в шаблоне. Если вам нужна изменяемая ссылка,
то `ref mut` будет работать аналогичным образом:

```rust
let mut x = 5;

match x {
    ref mut mr => println!("Получили изменяемую ссылку на {}", mr),
}
```

# Сопоставление с диапазоном

Вы можете сопоставлять с диапазоном значений, используя `...`:

```rust
let x = 1;

match x {
    1 ... 5 => println!("от одного до пяти"),
    _ => println!("что угодно"),
}
```

Этот код напечатает `от одного до пяти`.

Диапазоны в основном используются с числами или одиночными символами (`char`).

```rust
let x = '💅';

match x {
    'а' ... 'и' => println!("ранняя буква"),
    'к' ... 'я' => println!("поздняя буква"),
    _ => println!("что-то ещё"),
}
```

Этот код напечатает `что-то ещё`.

# Связывание

Вы можете связать значение с именем с помощью символа `@`:

```rust
let x = 1;

match x {
    e @ 1 ... 5 => println!("получили элемент диапазона {}", e),
    _ => println!("что угодно"),
}
```

Этот код напечатает `получили элемент диапазона 1`. Это полезно, когда вы хотите
сделать сложное сопоставление для части структуры данных:

```rust
#[derive(Debug)]
struct Person {
    name: Option<String>,
}

let name = "Steve".to_string();
let mut x: Option<Person> = Some(Person { name: Some(name) });
match x {
    Some(Person { name: ref a @ Some(_), .. }) => println!("{:?}", a),
    _ => {}
}
```

Этот код напечатает `Some("Steve")`: мы связали внутреннюю `name` с `a`.

Если вы используете `@` совместно с `|`, то вы должны убедиться, что имя
связывается в каждой из частей шаблона:

```rust
let x = 5;

match x {
    e @ 1 ... 5 | e @ 8 ... 10 => println!("получили элемент диапазона {}", e),
    _ => println!("что угодно"),
}
```

# Ограничители шаблонов

Вы можете ввести *ограничители шаблонов* (*match guards*) с помощью `if`:

```rust
enum OptionalInt {
    Value(i32),
    Missing,
}

let x = OptionalInt::Value(5);

match x {
    OptionalInt::Value(i) if i > 5 => println!("Получили целое больше пяти!"),
    OptionalInt::Value(..) => println!("Получили целое!"),
    OptionalInt::Missing => println!("Неудача."),
}
```

Этот код напечатает `Получили целое!`.

Если вы используете `if` с несколькими шаблонами, он применяется к обоим частям:

```rust
let x = 4;
let y = false;

match x {
    4 | 5 if y => println!("да"),
    _ => println!("нет"),
}
```

Этот код печатает `нет`, потому что `if` применяется ко всему `4 | 5`, а не
только к `5`. Другими словами, приоритет `if` выглядит так:

```text
(4 | 5) if y => ...
```

а не так:

```text
4 | (5 if y) => ...
```

# Заключение

Вот так! Существует много разных способов использования конструкции
сопоставления с шаблоном, и все они могут быть смешаны и состыкованы, в
зависимости от того, что вы хотите сделать:

```rust,ignore
match x {
    Foo { x: Some(ref name), y: None } => ...
}
```

Шаблоны — это очень мощный инструмент. Используйте их.
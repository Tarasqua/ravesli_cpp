# Глава №8.
## Урок №120. Введение в ООП
**Объекты имеют два основных компонента:**
* свойства;
* поведение.

**Объектно-ориентированное программирование** (сокр. **"ООП"**) предоставляет\
возможность создавать объекты, которые объединяют свойства и поведение в\
самостоятельный союз, который затем можно многоразово использовать.

## Урок №121. Классы, Объекты и Методы
### Классы
```c++
class DateClass {
    public:
        int m_day; // "m" = "members" - член класса
        int m_month;
        int m_year;
};

DateClass today {27, 9, 2023}; // экземпляр (объект) класса
```

### Методы классов
Функции, определенные внутри класса, называются **методами**.
```c++
#include <iostream>

class DateClass {
public:
    int m_day;
    int m_month;
    int m_year;

    void print() {
        std::cout << m_day << "/" << m_month << "/" << m_year;
    }
};

int main() {
    DateClass today {27, 9, 2023};
    today.m_day = 28;
    today.print(); // 28/9/2023
}
```

**Правило: Пишите имена классов с заглавной буквы.**

```c++
#include <iostream>
#include <string>

class Employee {
public:
    std::string m_name;
    int m_id;
    double m_wage;

    // Метод вывода информации о работнике на экран
    void print() {
        std::cout << "Name: " << m_name <<
                  "\nId: " << m_id <<
                  "\nWage: $" << m_wage << '\n';
    }
};

int main() {
    // Определяем двух работников
    Employee john{"John", 5, 30.00};
    Employee max{"Max", 6, 32.75};

    // Выводим информацию о работниках на экран
    john.print(); // Name: John	Id: 5	Wage: $30
    std::cout << std::endl;
    max.print(); // Name: Max	Id: 6	Wage: $32.75

    return 0;
}
```

### Примечание о структурах в C++
**Правило: Используйте ключевое слово `struct` для структур, используемых\
только для хранения данных. Используйте ключевое слово `class` для объектов,\
объединяющих как данные, так и функции.**

### Тест
**Задание №1.**
```c++
#include <iostream>

class Numbers {
public:
    int m_first;
    int m_second;

    void set(int first, int second) {
        m_first = first;
        m_second = second;
    }

    void print() {
        std::cout << "Numbers(" << m_first << ", " << m_second << ")\n";
    }
};

int main() {
    Numbers n1{};
    n1.set(3, 3);
    Numbers n2{4, 4};
    n1.print();
    n2.print();
    return 0;
}
```

**Задание №2.**\
**Почему для `Numbers` должен использоваться класс, а не структура?**\
Класс Numbers содержит как переменные-члены, так и методы, поэтому мы\
должны использовать класс. Мы не должны использовать структуры с\
объектами, которые имеют методы.

## Урок №122. Спецификаторы доступа `public` и `private`
**Открытые члены** (или **"public-члены"**) — это члены структуры или класса,\
к которым можно получить доступ извне этой же структуры или класса.

**Закрытые члены** (или **"private-члены"**) — это члены класса, доступ к которым\
имеют только другие члены этого же класса. Все члены класса являются закрытыми\
по умолчанию.

Ключевое слово `public` вместе с двоеточием называется спецификатором доступа.\
**Спецификатор доступа** определяет, кто имеет доступ к членам этого спецификатора.

**В языке C++ есть 3 уровня доступа:**
* **спецификатор `public`** делает члены открытыми;
* **спецификатор `private`** делает члены закрытыми;
* **спецификатор `protected`** открывает доступ к членам только для\
  дружественных и дочерних классов.

### Использование спецификаторов доступа
**Правило: Устанавливайте спецификатор доступа `private` переменным-членам\
класса и спецификатор доступа `public` — методам класса (если у вас нет веских\
оснований делать иначе).**

```c++
#include <iostream>

class DateClass {
    int m_day;
    int m_month;
    int m_year;

public:
    void setDate(int day, int month, int year) {
        m_day = day;
        m_month = month;
        m_year = year;
    }

    void print() const {
        std::cout << m_day << "/" << m_month << "/" << m_year;
    }
};

int main() {
    DateClass today;
    today.setDate(27, 9, 2023);
    today.print(); // 27/9/2023
    
    return 0;
}
```

**Контроль доступа работает на основе класса**, а не на основе объекта.\
Это означает, что, когда метод имеет доступ к закрытым членам класса,\
он может обращаться к закрытым членам любого объекта этого класса:
```c++
#include <iostream>

class DateClass {
    int m_day;
    int m_month;
    int m_year;

public:
    void setDate(int day, int month, int year) {
        m_day = day;
        m_month = month;
        m_year = year;
    }

    void print() const {
        std::cout << m_day << "/" << m_month << "/" << m_year;
    }
    
    // copyFrom() может не только напрямую обращаться к закрытым членам неявного
    // объекта с которым работает (копия объекта), но и имеет прямой доступ к закрытым
    // членам объекта b класса DateClass
    void copyFrom(const DateClass &b) {
        m_day = b.m_day;
        m_month = b.m_month;
        m_year = b.m_year;
    }
};

int main() {
    DateClass today;
    today.setDate(27, 9, 2023);
    today.print(); // 27/9/2023
    DateClass copy;
    copy.copyFrom(today); // 27/9/2023
    copy.print();
    
    return 0;
}
```

### Структуры vs. Классы
**Класс по умолчанию устанавливает всем своим членам спецификатор доступа\
`private`. Структура же по умолчанию устанавливает всем своим членам\
спецификатор доступа `public`.**

**Структуры наследуют от других конструкций языка С++ открыто, в то время как\
классы наследуют закрыто.**

### Тест
**Ответ №1. а)**\
**Открытый член** — это член класса, доступ к которому имеют объекты как внутри,\
так и извне класса.

**Ответ №1. b)**\
**Закрытый член** — это член класса, доступ к которому имеют только другие члены\
этого же класса.

**Ответ №1. c)**\
**Спецификатор доступа** определяет, кто имеет доступ к членам этого же\
спецификатора.

**Ответ №1. d)**\
В языке С++ есть **3 спецификатора доступа**:
* public;
* private;
* protected.

**Задание №2. a)**
```c++
#include <iostream>

class Numbers {
private:
    double m_a, m_b, m_c;
public:
    void setValues(double a, double b, double c) {
        m_a = a;
        m_b = b;
        m_c = c;
    }

    void print() const {
        std::cout << "<" << m_a << ", " << m_b << ", " << m_c << ">";
    }
};

int main() {
    Numbers point{};
    point.setValues(3.0, 4.0, 5.0);
    point.print(); // <3, 4, 5>
    return 0;
}
```

**Задание №2. b)**
```c++
#include <iostream>

class Numbers {
private:
    double m_a, m_b, m_c;
public:
    void setValues(double a, double b, double c) {
        m_a = a;
        m_b = b;
        m_c = c;
    }

    void print() {
        std::cout << "<" << m_a << ", " << m_b << ", " << m_c << ">";
    }

    // Здесь мы можем использовать тот факт, что контроль доступа осуществляется на
    // основе класса для того, чтобы получить доступ напрямую к закрытым членам
    // объекта d класса Numbers
    bool isEqual(const Numbers &d) {
        return (m_a == d.m_a && m_b == d.m_b && m_c == d.m_c);
    }
};

int main() {
    Numbers point1{};
    point1.setValues(3.0, 4.0, 5.0);
    Numbers point2{};
    point2.setValues(3.0, 4.0, 5.0);

    if (point1.isEqual(point2))
        std::cout << "point1 and point2 are equal\n"; // point1 and point2 are equal
    else
        std::cout << "point1 and point2 are not equal\n";

    Numbers point3{};
    point3.setValues(7.0, 8.0, 9.0);
    
    if (point1.isEqual(point3))
        std::cout << "point1 and point3 are equal\n";
    else
        std::cout << "point1 and point3 are not equal\n"; // point1 and point3 are not equal
    return 0;
}
```

**Задание №3.**\
Функционал стека.
```c++
#include <iostream>
#include <cassert>

class Stack {
private:
    int m_array[10]; // это будут данные нашего стека
    int m_next; // это будет индексом следующего свободного элемента стека
public:
    void reset() {
        m_next = 0;
        for (int i = 0; i < 10; ++i)
            m_array[i] = 0;
    }

    bool push(int value) {
        // Если стек уже заполнен, то возвращаем false
        if (m_next == 10)
            return false;
        // присваиваем следующему свободному элементу значение value, а затем увеличиваем m_next
        m_array[m_next++] = value;
        return true;
    }

    int pop() {
        // Если элементов в стеке нет, то выводим стейтмент assert
        assert (m_next > 0);
        // m_next указывает на следующий свободный элемент, поэтому последний элемент со значением - это m_next-1.
        // Мы хотим сделать следующее:
        // int val = m_array[m_next-1];
        // получаем последний элемент со значением --m_next;
        // m_next теперь на единицу меньше, так как мы только что вытянули верхний элемент стека
        // return val; // возвращаем элемент
        // Весь вышеприведенный код можно заменить следующей (одной) строкой кода
        return m_array[--m_next];
    }

    void print() {
        std::cout << "( ";
        for (int i = 0; i < m_next; ++i)
            std::cout << m_array[i] << ' ';
        std::cout << ")\n";
    }
};

int main() {
    Stack stack{};
    stack.reset();
    stack.print(); // ( )
    stack.push(3);
    stack.push(7);
    stack.push(5);
    stack.print(); // (3 7 5)
    stack.pop();
    stack.print(); // (3 7)
    stack.pop();
    stack.pop();
    stack.print(); // ( )
    return 0;
}
```

## Урок №123. Инкапсуляция, Геттеры и Сеттеры
### Инкапсуляция
В объектно-ориентированном программировании **инкапсуляция** (или **"сокрытие\
информации"**) — это процесс скрытого хранения деталей реализации объекта.\
Пользователи обращаются к объекту через открытый интерфейс.

**Преимущества инкапсуляции:**
1. инкапсулированные классы проще в использовании и уменьшают сложность\
   ваших программ;
2. инкапсулированные классы помогают защитить ваши данные и предотвращают\
   их неправильное использование;
3. инкапсулированные классы легче изменить;
4. c инкапсулированными классами легче проводить отладку.

### Функции доступа (геттеры и сеттеры)
**Функция доступа** — это короткая открытая функция, задачей которой является\
получение или изменение значения закрытой переменной-члена класса:

**Функции доступа обычно бывают двух типов:**
* **геттеры** — это функции, которые возвращают значения закрытых\
  переменных-членов класса;
* **сеттеры** — это функции, которые позволяют присваивать значения закрытым\
  переменным-членам класса.

```c++
class Date {
private:
    int m_day;
    int m_month;
    int m_year;

public:
    int getDay() { return m_day; } // геттер для day
    void setDay(int day) { m_day = day; } // сеттер для day

    int getMonth() { return m_month; } // геттер для month
    void setMonth(int month) { m_month = month; } // сеттер для month

    int getYear() { return m_year; } // геттер для year
    void setYear(int year) { m_year = year; } // сеттер для year
};
```

**Правило: Предоставляйте функции доступа только в том случае, когда нужно,\
чтобы пользователь имел возможность получать или присваивать значения\
членам класса.**

**Правило: Геттеры должны использовать тип возврата по значению или по\
константной ссылке. Не используйте для геттеров тип возврата по неконстантной\
ссылке.**

```c++
#include <iostream>

class Date {
private:
    int m_day;
    int m_month;
    int m_year;

public:
    int &getDay() { return m_day; } // геттер для day
    void setDay(int day) { m_day = day; } // сеттер для day

    int &getMonth() { return m_month; } // геттер для month
    void setMonth(int month) { m_month = month; } // сеттер для month

    int &getYear() { return m_year; } // геттер для year
    void setYear(int year) { m_year = year; } // сеттер для year
};

int main() {
    Date today{};
    today.setDay(27);
    today.setMonth(9);
    today.setYear(2023);

    auto &day_today = today.getDay();
    std::cout << day_today; // 27
    today.setDay(30);
    std::cout << day_today; // 30

    return 0;
}
```

## Урок №124. Конструкторы
**Конструктор** — это особый тип метода класса, который автоматически вызывается\
при создании объекта этого же класса. Конструкторы обычно используются для\
инициализации переменных-членов класса значениями, которые предоставлены по\
умолчанию/пользователем, или для выполнения любых шагов настройки,\
необходимых для используемого класса.

В отличие от обычных методов, **конструкторы имеют определенные правила их\
именования:**
* конструкторы всегда должны иметь то же имя, что и класс (учитываются\
  верхний и нижний регистры);
* конструкторы не имеют типа возврата (даже `void`).

### Конструкторы по умолчанию
Конструктор, который не имеет параметров (или содержит параметры, которые все\
имеют значения по умолчанию), называется **конструктором по умолчанию**. Он\
вызывается, если пользователем не указаны значения для инициализации.
```c++
#include <iostream>

class Fraction {
private:
    int m_numerator;
    int m_denominator;
public:
    Fraction() { // конструктор по умолчанию
        m_numerator = 0;
        m_denominator = 1;
    }
    int &getNumerator() { return m_numerator; }
    int &getDenominator() { return m_denominator; }
    double getValue() {
        return static_cast<double>(m_numerator) /
               m_denominator;
    }
};

int main() {
    Fraction drob; // так как нет никаких аргументов, то вызывается конструктор по умолчанию Fraction()
    std::cout << drob.getNumerator() << "/" << drob.getDenominator() << '\n'; // 0/1

    return 0;
}
```

### Конструкторы с параметрами
```c++
#include <iostream>
#include <cassert>

class Fraction {
private:
    int m_numerator;
    int m_denominator;
public:
    Fraction() { // конструктор по умолчанию
        m_numerator = 0;
        m_denominator = 1;
    }

    // Конструктор с двумя параметрами, один из которых имеет значение по умолчанию
    Fraction(int numerator, int denominator = 1) {
        assert(denominator != 0);
        m_numerator = numerator;
        m_denominator = denominator;
    }

    int &getNumerator() { return m_numerator; }

    int &getDenominator() { return m_denominator; }

    double getValue() {
        return static_cast<double>(m_numerator) /
               m_denominator;
    }
};

int main() {
    Fraction drob{1, 2};
    std::cout << drob.getNumerator() << "/" << drob.getDenominator() << '=' << drob.getValue() << '\n'; // 1/2=0.5

    return 0;
}
```

**Правило: Используйте прямую инициализацию или uniform-инициализацию с\
объектами ваших классов.**

### Копирующая инициализация
```c++
int a = 7; // копирующая инициализация
Fraction eight = Fraction(8); // копирующая инициализация, вызывается Fraction(8,1)
Fraction nine = 9; // копирующая инициализация. Компилятор будет искать пути
// конвертации 9 в Fraction, что приведет к вызову конструктора Fraction(9, 1)
```

**Правило: Не используйте копирующую инициализацию с объектами ваших классов.**

### Неявно генерируемый конструктор по умолчанию
Если ваш класс не имеет конструкторов, то язык C++ автоматически сгенерирует для\
вашего класса открытый конструктор по умолчанию. Его иногда называют **неявным\
конструктором** (или **"неявно сгенерированным конструктором"**):
```c++
class Date
{
private:
    int m_day = 12;
    int m_month = 1;
    int m_year = 2018;
public:
    Date() { // неявно генерируемый конструктор
    }
    // Или проще
    Date() = default;
};
```

**Правило: Создавайте хотя бы один конструктор в классе, даже если это пустой\
конструктор по умолчанию.**

### Классы, содержащие другие классы
```c++
#include <iostream>
#include <cassert>

class A {
public:
    A() { std::cout << "A\n"; }
};

class B {
private:
    A m_a; // B содержит A, как переменную-член
public:
    B() { std::cout << "B\n"; }
};


int main() {
    B b; // A   B
    return 0;
}
```
При создании переменной `b` вызывается конструктор B(). Прежде чем тело\
конструктора выполнится, `m_a` инициализируется, вызывая конструктор по\
умолчанию класса A. Таким образом выведется `A`. Затем управление возвратится\
обратно к конструктору B, и тело конструктора B начнет свое выполнение.

### Тест
**Задание №1, a).**
```c++
#include <iostream>
#include <string>

class Ball {
private:
    std::string m_color;
    double m_radius;
public:
    // Конструктор по умолчанию без параметров
    Ball() {
        m_color = "red";
        m_radius = 20.0;
    }
    // Конструктор с параметром color (для radius предоставлено значение по умолчанию)
    Ball(const std::string &color) {
        m_color = color;
        m_radius = 20.0;
    }
    // Конструктор с параметром radius (для color предоставлено значение по умолчанию)
    Ball(double radius) {
        m_color = "red";
        m_radius = radius;
    }
    // Конструктор с параметрами color и radius
    Ball(const std::string &color, double radius) {
        m_color = color;
        m_radius = radius;
    }

    void print() {
        std::cout << "color: " << m_color << ", radius: " << m_radius << '\n';
    }
};

int main() {
    Ball def;
    def.print(); // color: red, radius: 20
    Ball black("black");
    black.print(); // color: black, radius: 20
    Ball thirty(30.0);
    thirty.print(); // color: red, radius: 30
    Ball blackThirty("black", 30.0);
    blackThirty.print(); // color: black, radius: 30
    return 0;
}
```

**Задание №1, b).**
```c++
#include <iostream>
#include <string>

class Ball {
private:
    std::string m_color;
    double m_radius;
public:
    // Конструктор с параметром radius (для color предоставлено значение по умолчанию)
    Ball(double radius) {
        m_color = "red";
        m_radius = radius;
    }
    // Конструктор с параметрами color и radius.
    // Обрабатываются такие случаи, как не предоставлено никаких
    // параметров, только color, color + radius
    Ball(const std::string &color="red", double radius=20.0) {
        m_color = color;
        m_radius = radius;
    }

    void print() {
        std::cout << "color: " << m_color << ", radius: " << m_radius << '\n';
    }
};

int main() {
    Ball def;
    def.print(); // color: red, radius: 20
    Ball black("black");
    black.print(); // color: black, radius: 20
    Ball thirty(30.0);
    thirty.print(); // color: red, radius: 30
    Ball blackThirty("black", 30.0);
    blackThirty.print(); // color: black, radius: 30
    return 0;
}
```

**Задание №2.**
Если не определить каких-либо конструкторов, то компилятор автоматически\
создаст пустой открытый конструктор по умолчанию. Если определить хотя бы\
один любой конструктор (по умолчанию или другой), то компилятор не создаст\
пустой конструктор по умолчанию.

## Урок №125. Список инициализации членов класса
Метод инициализации переменных-членов класса через **список инициализации членов**,\
вместо присваивания им значений после объявления:
```c++
#include <iostream>

class Values {
private:
    int m_value1;
    double m_value2;
    char m_value3;
public:
    // напрямую инициализируем переменные-члены класса
    Values() : m_value1{3}, m_value2{4.5}, m_value3{'d'} {
        // Нет необходимости использовать присваивание
    }

    void print() const {
        std::cout << "Values(" << m_value1 << ", " << m_value2 << ", " << m_value3 << ")\n";
    }
};

int main() {
    Values value;
    value.print(); // Values(3, 4.5, d)

    return 0;
}
```

Можно также добавить возможность caller-у передавать значения для\
инициализации:
```c++
#include <iostream>

class Values {
private:
    int m_value1;
    double m_value2;
    char m_value3;
public:
    // напрямую инициализируем переменные-члены класса
    Values(int value1, double value2, char value3 = 'd')
            : m_value1{value1}, m_value2{value2}, m_value3{value3} {
        // Нет необходимости использовать присваивание
    }

    void print() const {
        std::cout << "Values(" << m_value1 << ", " << m_value2 << ", " << m_value3 << ")\n";
    }
};

int main() {
    Values value{3, 4.5};
    value.print(); // Values(3, 4.5, d)

    return 0;
}
```

Мы можем использовать параметры по умолчанию для предоставления значений\
по умолчанию, если пользователь их не предоставил. Например, класс, который\
имеет константную переменную-член:
```c++
class AnotherValues {
private:
    const int m_value;
public:
    AnotherValues() : m_value{7} { // напрямую инициализируем константную переменную-член
    }
};
```

**Правило: Используйте списки инициализации членов, вместо операций\
присваивания, для инициализации переменных-членов вашего класса.**

**Правило: Используйте uniform-инициализацию вместо прямой инициализации.**

### Инициализация массивов в классе
```c++
class Values {
private:
    const int m_array[7];
public:
    // uniform-инициализация для инициализации массива
    Values() : m_array{1, 2, 3, 4, 5, 6, 7} { 
    }
};
```

### Инициализация переменных-членов, которые являются классами
```c++
#include <iostream>

class A {
public:
    A(int a) { std::cout << "A " << a << "\n"; }
};

class B {
private:
    A m_a;
public:
    B(int b) : m_a(b - 1) { // вызывается конструктор A(int) для инициализации члена m_a
        std::cout << "B " << b << "\n";
    }
};

int main() {
    B b(7); // A 6    B 7
    return 0; 
}
```
При создании переменной `b` вызывается конструктор `B(int)` со значением `7`. До\
того, как тело конструктора выполнится, инициализируется `m_a`, вызывая\
конструктор `A(int)` со значением `6`. Таким образом, выведется `A 6`. Затем\
управление возвратится обратно к конструктору B(), и тогда уже он выполнится и\
выведется `B 7`.

### Использование списков инициализации
Если список инициализации помещается на той же строке, что и имя конструктора,\
то лучше всё разместить в одной строке:
```c++
class Values {
private:
    int m_value1;
    double m_value2;
    char m_value3;
public:
    Values() : m_value1{3}, m_value2{4.5}, m_value3{'d'} // всё находится в одной строке
    {
    }
};
```

Если список инициализации членов не помещается в строке с именем конструктора,\
то на следующей строке (используя перенос) инициализаторы должны быть с\
отступом:
```c++
class Values {
private:
    int m_value1;
    double m_value2;
    char m_value3;
public:
    Values(int value1, double value2, char value3 = 'd') // на этой строке и так уже много всего
            : m_value1{value1}, m_value2{value2}, m_value3{value3} // переносим инициализацию на новую строку
    {
    }
};
```

Или даже так:
```c++
class Values {
private:
    int m_value1;
    double m_value2;
    char m_value3;
    float m_value4;
public:
    Values(int value1, double value2, char value3 = 'd', float value4 = 17.5) // на этой строке и так уже много всего
            : m_value1{value1},
              m_value2{value2},
              m_value3{value3},
              m_value4{value4} {
    }
};
```

### Порядок выполнения в списке инициализации
**Переменные в списке инициализации инициализируются в том порядке,\
в котором объявлены в классе.**

### Резюме
Списки инициализации членов позволяют инициализировать члены, а не\
присваивать им значения. Это единственный способ инициализации констант и\
ссылок, которые являются переменными-членами вашего класса. Во многих случаях\
использование списка инициализации может быть более результативным, чем\
присваивание значений переменным-членам в теле конструктора. Списки\
инициализации работают как с переменными фундаментальных типов данных, так\
и с членами, которые сами являются классами.

### Тест
```c++
#include <iostream>

class RGBA {
private:
    std::uint8_t m_red;
    std::uint8_t m_green;
    std::uint8_t m_blue;
    std::uint8_t m_alpha;
public:
    explicit RGBA(std::uint8_t red = 0, std::uint8_t green = 0, std::uint8_t blue = 0,
         std::uint8_t alpha = 255) :
            m_red(red), m_green(green), m_blue(blue), m_alpha(alpha) {
    }

    void print() const {
        std::cout << "r=" << static_cast<int>(m_red) <<
                  " g=" << static_cast<int>(m_green) <<
                  " b=" << static_cast<int>(m_blue) <<
                  " a=" << static_cast<int>(m_alpha) << '\n';
    }
};

int main() {
    RGBA color{0, 135, 135};
    color.print(); // r=0 g=135 b=135 a=255
    return 0;
}
```

### Урок №126. Инициализация нестатических членов класса
```c++
#include <iostream>

class Something {
private:
    double m_length = 3.5;
    double m_width = 3.5;

public:
    Something()= default;
    // m_length и m_width инициализируются конструктором
    // (значения по умолчанию, приведенные выше, не используются)
    Something(double length, double width)
            : m_length(length), m_width(width) {
    }

    void print() const {
        std::cout << "length: " << m_length << " and width: " << m_width << '\n';
    }
};

int main() {
    Something a(4.5, 5.5);
    a.print(); // length: 4.5 and width: 5.5
    Something b;
    b.print(); // length: 3.5 and width: 3.5

    return 0;
}
```

**Правило: Используйте инициализацию нестатических членов для указания\
значений по умолчанию переменным-членам.**

## Урок №127. Делегирующие конструкторы
### Решение в C++11
```c++
class Boo {
public:
    Boo() {
        // Часть кода X
    }
    Boo(int value) : Boo() // используем конструктор, указанный выше, для выполнения части кода X
    {
        // Часть кода Y
    }
};
```

### Использование отдельного метода
```c++
class Boo {
private:
    void DoX() {
        // Часть кода Х
    }
public:
    Boo() {
        DoX();
    }
    Boo(int value)
    {
        DoX();
        // Часть кода Y
    }
};
```

Функция `Init()` для инициализации переменных-членов обратно значениями по умолчанию,\
а затем каждый конструктор вызывает функцию `Init()` перед своим фактическим выполнением.\
Это сокращает дублирование кода до минимума и позволяет явно вызывать `Init()` из любого\
места в программе:
```c++
class Boo {
public:
    Boo() {
        Init();
    }

    Boo(int value)
    {
        Init();
        // Часть кода Y
    }

    void Init() {
        // Код для инициализации Boo
    }
};
```

### Делегирующие конструкторы в C++11
Начиная с C++11, конструкторам разрешено вызывать другие конструкторы. Этот\
процесс называется **делегированием конструкторов** (или **"цепочкой конструкторов"**).\
Чтобы один конструктор вызывал другой, нужно просто сделать вызов этого\
конструктора в списке инициализации членов:
```c++
class Boo {
private:
public:
    Boo() {
        // Часть кода X
    }

    Boo(int value) : Boo() // используем конструктор по умолчанию Boo() для выполнения части кода X
    {
        // Часть кода Y
    }
};
```

Конкретный пример:
```c++
#include <iostream>
#include <string>

class Employee {
private:
    int m_id;
    std::string m_name;
public:
    Employee(int id = 0, const std::string &name = "") :
            m_id(id), m_name(name) {
        std::cout << "Employee " << m_name << " created.\n";
    }

    // Используем делегирующие конструкторы для сокращения дублированного кода
    Employee(const std::string &name) : Employee(0, name) {}
};

int main() {
    Employee a; // Employee  created.
    Employee b("Ivan"); // Employee Ivan created.

    return 0;
}
```

### Несколько заметок о делегирующих конструкторах
**Конструкторы могут либо вызывать другие конструкторы, либо выполнять\
инициализацию, но не всё сразу.**

**Один конструктор может вызывать другой конструктор, в коде которого\
может находиться вызов первого конструктора. Это создаст бесконечный цикл и\
приведет к тому, что память стека закончится и произойдет сбой.**

## Урок №128. Деструкторы
**Деструктор** — это специальный тип метода класса, который выполняется при\
удалении объекта класса.

### Имена деструкторов
Так же, как и конструкторы, деструкторы имеют свои правила, которые\
касаются их имен:
* деструктор должен иметь то же имя, что и класс, со знаком тильда (`~`) в\
  самом начале;
* деструктор не может принимать аргументы;
* деструктор не имеет типа возврата.

Из второго правила вытекает еще одно правило: для каждого класса может\
существовать только один деструктор, так как нет возможности перегрузить\
деструкторы, как функции, и отличаться друг от друга аргументами они не могут.

### Пример использования деструктора на практике
```c++
#include <iostream>
#include <cassert>

class Massiv {
private:
    int *m_array; // динамическое выделение, так как не знаем длину на момент создания
    int m_length;
public:
    Massiv(int length) { // конструктор
        assert(length > 0);

        m_array = new int[length];
        m_length = length;
    }

    ~Massiv() { // деструктор
        // Динамически удаляем массив, который выделили ранее
        delete[] m_array;
    }

    void setValue(int index, int value) { m_array[index] = value; }

    int &getValue(int index) { return m_array[index]; }

    int &getLength() { return m_length; }
};

int main() {
    Massiv arr(15); // выделяем 15 целочисленных значений
    for (int count = 0; count < 15; ++count)
        arr.setValue(count, count + 1);

    std::cout << "The value of element 7 is " << arr.getValue(7); // The value of element 7 is 8

    return 0;
} // объект arr удаляется здесь, поэтому деструктор ~Massiv() вызывается тоже здесь

```

### Выполнение конструкторов и деструкторов
```c++
#include <iostream>

class Another {
private:
    int m_nID;
public:
    Another(int nID) {
        std::cout << "Constructing Another " << nID << '\n';
        m_nID = nID;
    }

    ~Another() {
        std::cout << "Destructing Another " << m_nID << '\n';
    }

    int getID() { return m_nID; }
};

int main() {
    // Выделяем объект класса Another из стека
    Another object(1); // Constructing Another 1
    std::cout << object.getID() << '\n'; // 1

    // Выделяем объект класса Another динамически из кучи
    Another *pObject = new Another(2); // Constructing Another 2
    std::cout << pObject->getID() << '\n'; // 2
    delete pObject; // Destructing Another 2

    return 0;
} // объект object выходит из области видимости здесь 
// Destructing Another 1
```

Обратите внимание, `Another 1` уничтожается после `Another 2`, так как мы\
удалили `pObject` до завершения выполнения функции main(), тогда как объект\
`object` не был удален до конца main().

### Идиома программирования RAII
**Идиома RAII** (англ. «**R**esource **A**cquisition **I**s **I**nitialization» = «Получение ресурсов есть\
инициализация») — это идиома объектно-ориентированного программирования,\
при которой использование ресурсов привязывается к времени жизни объектов с\
автоматической продолжительностью жизни.

**Правило: Используйте идиому программирования RAII и не выделяйте объекты\
вашего класса динамически.**

### Предупреждение о функции `exit()`
**Если вы используете функцию `exit()`, то ваша программа завершится, и никакие\
деструкторы не будут вызваны.**

### Резюме 
Используя конструкторы и деструкторы, ваши классы могут выполнять\
инициализацию и очистку после себя автоматически без вашего участия! Это\
уменьшает вероятность возникновения ошибок и упрощает процесс использования\
классов.

## Урок №129. Скрытый указатель `*this`
### Скрытый указатель `*this`
**Указатель `*this`** — это скрытый константный указатель, содержащий адрес\
объекта, который вызывает метод класса.

### Указатель `*this` всегда указывает на текущий объект
Каждый метод имеет в качестве параметра указатель `*this`, который указывает на\
адрес объекта, с которым в данный момент выполняется операция, например:
```c++
int main() {
    Another X(3); // *this = &X внутри конструктора Another
    Another Y(4); // *this = &Y внутри конструктора Another
    X.setNumber(5); // *this = &X внутри метода setNumber
    Y.setNumber(6); // *this = &Y внутри метода setNumber
    
    return 0;
}
```

### Явное указание указателя `*this`
Если у вас есть конструктор (или метод), который имеет параметр с тем же именем,\
что и переменная-член, то устранить неоднозначность можно с помощью указателя `*this`:
```c++
class Something {
private:
    int data;

public:
    Something(int data) { // конструктор принимает параметр с тем же именем, что и переменная-член
        this->data = data; // явно указываем на переменную-член
    }
};
```

### Цепочки методов класса
```c++
#include <iostream>

class Mathem {
private:
    int m_value;
public:
    Mathem() { m_value = 0; }

    Mathem& add(int value) { m_value += value; }

    Mathem& sub(int value) { m_value -= value; }

    Mathem& multiply(int value) { m_value *= value; }

    int &getValue() { return m_value; }
};

int main() {
    Mathem operation;
    operation.add(7).sub(5).multiply(3);
    std::cout << operation.getValue() << '\n'; // 6

    return 0;
}
```

Рассмотрим детально:
* Сначала вызывается `operation.add(7)`, который добавляет `7`\
  к нашему `m_value`.
* Затем `add()` возвращает указатель `*this`, который является\ 
  ссылкой на объект `operation`.
* Затем вызов `operation.sub(5)` вычитает `5` из `m_value` и\
  возвращает `operation`.
* `multiply(3)` умножает `m_value` на `3` и возвращает `operation`,\
  который уже игнорируется.
* Однако, поскольку каждая функция модифицировала `operation`, `m_value`\
  объекта `operation` теперь содержит значение `((0 + 7) - 5) * 3)`,\
  которое равно `6`.

### Резюме
Указатель *this является скрытым параметром, который неявно добавляется к\
каждому методу класса. В большинстве случаев нам не нужно обращаться к нему\
напрямую, но при необходимости это можно сделать. Стоит отметить, что указатель\
`*this` является константным указателем — вы можете изменить значение исходного\
объекта, но вы не можете заставить указатель *this указывать на что-то другое!\

Если у вас есть функции, которые возвращают `void`, то возвращайте `*this` вместо\
`void`. Таким образом, вы сможете соединить несколько методов в одну «цепочку».

## Урок №130. Классы и заголовочные файлы
**Пример:** класс `Date` с конструктором `Date()` и методом `setDate()`,\
определенными вне тела класса. Обратите внимание, прототипы этих функций\
все еще находятся внутри тела класса, но их фактическая реализация находится\
за его пределами:
```c++
class Date {
private:
    int m_day{};
    int m_month{};
    int m_year{};
public:
    Date(int day, int month, int year);

    void SetDate(int day, int month, int year);

    int &getDay() { return m_day; }

    int &getMonth() { return m_month; }

    int &getYear() { return m_year; }
};

// Конструктор класса Date
Date::Date(int day, int month, int year) {
    SetDate(day, month, year);
}

// Метод класса Date
void Date::SetDate(int day, int month, int year) {
    m_day = day;
    m_month = month;
    m_year = year;
}
```

Еще пример:
```c++
class Mathem {
private:
    int m_value = 0;
public:
    Mathem(int value = 0);

    Mathem &add(int value);
    Mathem &sub(int value);
    Mathem &divide(int value);
    int &getValue() { return m_value; }
};

Mathem::Mathem(int value) : m_value(value) {
}

Mathem &Mathem::add(int value) {
    m_value += value;
    return *this;
}

Mathem &Mathem::sub(int value) {
    m_value -= value;
    return *this;
}

Mathem &Mathem::divide(int value) {
    m_value /= value;
    return *this;
}
```

### Классы и заголовочные файлы
**date.h:**
```c++
#ifndef RAVESLI_DATE_H
#define RAVESLI_DATE_H

class Date {
private:
    int m_day{};
    int m_month{};
    int m_year{};
public:
    Date(int day, int month, int year);
    void SetDate(int day, int month, int year);
    int &getDay() { return m_day; }
    int &getMonth() { return m_month; }
    int &getYear() { return m_year; }
};

#endif //RAVESLI_DATE_H
```

**date.cpp:**
```c++
#include "date.h"

// Конструктор класса Date
Date::Date(int day, int month, int year) {
    SetDate(day, month, year);
}

// Метод класса Date
void Date::SetDate(int day, int month, int year) {
    m_day = day;
    m_month = month;
    m_year = year;
}
```

```c++
#include <iostream>
#include "date.h"

int main() {
    Date date{21, 1, 2023};
    std::cout << date.getDay(); // 21

    return 0;
}
```

**Параметры по умолчанию для методов должны быть объявлены в теле класса (в\
заголовочном файле), где они будут видны всем, кто подключает этот заголовочный\
файл с классом.**

### Резюме
* Классы, используемые только в одном файле, и которые повторно не\
  используются, определяйте непосредственно в файле .cpp, где они\
  используются.
* Классы, используемые в нескольких файлах или предназначенные для\
  повторного использования, определяйте в заголовочном файле с тем же\
  именем, что у класса.
* Тривиальные методы (обычные конструкторы или деструкторы, функции\
  доступа и т.д.) определяйте внутри тела класса.
* Нетривиальные методы определяйте в файле .cpp с тем же именем, что у\
  класса.

## Урок №131. Классы и `const`

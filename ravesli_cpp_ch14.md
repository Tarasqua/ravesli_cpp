# Глава №14. Исключения в C++
## Содержание
1. [Урок №190. Обработка исключений. Операторы `throw`, `try` и `catch`](#урок-190-обработка-исключений-операторы-throw-try-и-catch)
2. [Урок №191. Исключения, Функции и Раскручивание стека](#урок-191-исключения-функции-и-раскручивание-стека)
3. [Урок №192. Непойманные исключения и обработчики `catch-all`](#урок-192-непойманные-исключения-и-обработчики-catch-all)
4. [Урок №193. Классы-Исключения и Наследование](#урок-193-классы-исключения-и-наследование)
5. [Урок №194. Повторная генерация исключений](#урок-194-повторная-генерация-исключений)
6. [Урок №195. Функциональный try-блок](#урок-195-функциональный-try-блок)
7. [Урок №196. Недостатки и опасности использования исключений](#урок-196-недостатки-и-опасности-использования-исключений)
8. [Глава №14. Итоговый тест](#глава-14-итоговый-тест)

## Урок №190. Обработка исключений. Операторы `throw`, `try` и `catch`
### Генерация исключений
Сигнализирование о том, что произошло исключение, называется **генерацией\
исключения** (или **"выбрасыванием исключения"**).

```c++
throw -1; // генерация исключения типа int
throw ENUM_INVALID_INDEX; // генерация исключения типа enum
throw "Can not take square root of negative number"; // генерация исключения типа const char* (строка C-style)
throw dX; // генерация исключения типа double (переменная типа double, которая была определена ранее)
throw MyException("Fatal Error"); // генерация исключения с использованием объекта класса MyException
```

### Поиск исключений
```c++
try
{
    // Здесь мы пишем стейтменты, которые будут генерировать следующее исключение
    throw -1; // типичный стейтмент throw
}
```

### Обработка исключений
**Ключевое слово `catch`** используется для определения блока кода (так называемого\
**«блока `catch`»**), который обрабатывает исключения определенного типа данных.

```c++
catch (int a)
{
    // Обрабатываем исключение типа int
    std::cerr << "We caught an int exception with value" << a << '\n';
}
```

```c++
catch (double) // примечание: Мы не указываем имя переменной, 
    // так как в этом нет надобности (мы её нигде в блоке не используем)
{
    // Обрабатываем исключение типа double здесь
    std::cerr << "We caught an exception of type double" << '\n';
}
```

### Использование `throw`, `try` и `catch` вместе
```c++
#include <iostream>
#include <string>

int main() {
    try {
        // Здесь мы пишем стейтменты, которые будут генерировать следующее исключение
        throw -1; // типичный стейтмент throw
    }
    catch (int a) {
        // Любые исключения типа int, сгенерированные в блоке try, приведенном выше, обрабатываются здесь
        std::cerr << "We caught an int exception with value: " << a << '\n';
    }
    catch (double) // мы не указываем имя переменной, так как в этом нет надобности (мы её нигде в блоке не используем)
    {
        // Любые исключения типа double, сгенерированные в блоке try, приведенном выше, обрабатываются здесь
        std::cerr << "We caught an exception of type double" << '\n';
    }
    catch (const std::string &str) // ловим исключения по константной ссылке
    {
        // Любые исключения типа std::string, сгенерированные внутри блока try, приведенном выше, обрабатываются здесь
        std::cerr << "We caught an exception of type std::string" << '\n';
    }

    std::cout << "Continuing our way!\n";

    return 0;
}
// We caught an int exception with value -1
// Continuing our way!
```

Оператор `throw` используется для генерации исключения `-1` типа `int`. Затем блок `try`\
обнаруживает оператор `throw` и перемещает его в соответствующий блок `catch`,\
который обрабатывает исключения типа `int`. Блок `catch` типа `int` и выводит\
соответствующее сообщение об ошибке.\
После обработки исключения, программа продолжает свое выполнение и выводит
на экран `Continuing our way!`.

### Исключения обрабатываются немедленно
```c++
#include <iostream>

int main() {
    try {
        throw 7.4; // выбрасывается исключение типа double
        std::cout << "This will never prints\n"; // не напечатается вовсе
    }
    catch (double a) { // обрабатывается исключение типа double
        std::cerr << "We caught a double of value: " << a << '\n';
    }

    return 0;
}

// We caught a double of value: 7.4
```

### Еще один пример
```c++
#include <iostream>
#include "math.h" // для функции sqrt()

int main() {
    std::cout << "Enter a number: ";
    double a;
    std::cin >> a; // -1

    try { // ищем исключения внутри этого блока и отправляем их в соответствующий обработчик catch
        // Если пользователь ввел отрицательное число, то выбрасывается исключение
        if (a < 0.0)
            throw "Can not take sqrt of negative number"; // выбрасывается исключение типа const char*

        // Если пользователь ввел положительное число, то выполняется операция и выводится результат
        std::cout << "The sqrt of " << a << " is " << sqrt(a) << '\n';
    }
    catch (const char *exception) { // обработчик исключений типа const char*
        std::cerr << "Error: " << exception << '\n'; // Error: Can not take sqrt of negative number
    }
}
```

## [Урок №191. Исключения, Функции и Раскручивание стека](#урок-191-исключения-функции-и-раскручивание-стека)
### Генерация исключений за пределами блока `try`
```c++
#include <cmath> // для sqrt()
#include <iostream>

// Отдельная функция вычисления квадратного корня
double mySqrt(double a) {
    // Если пользователь ввел отрицательное число, то выбрасываем исключение
    if (a < 0.0)
        throw "Can not take sqrt of negative number"; // выбрасывается исключение типа const char*

    return sqrt(a);
}

int main() {
    std::cout << "Enter a number: ";
    double a;
    std::cin >> a;

    try { // ищем исключения, которые выбрасываются в блоке try, и отправляем их для обработки в блок(и) catch
        double d = mySqrt(a);
        std::cout << "The sqrt of " << a << " is " << d << '\n';
    }
    catch (const char *exception) { // обработка исключений типа const char*
        std::cerr << "Error: " << exception << std::endl;
    }

    return 0;
}

// Enter a number: -3
// Error: Can not take sqrt of negative number
```

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/info.svg">
>   <img alt="Info" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/info.svg">
> </picture><br>
>
> Процесс, в результате которого программа последовательно покидает составные\
> инструкции и определения функций в поисках предложения `catch`, способного\
> обработать возникшее исключение, называется **раскруткой стека**.

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/info.svg">
>   <img alt="Info" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/info.svg">
> </picture><br>
>
> Блок `try` ловит исключения не только внутри себя, но и внутри функций,\
> которые вызываются в этом блоке `try`.

### Еще один пример раскручивания стека
```c++
#include <iostream>

void last() // вызывается функцией three()
{
    std::cout << "Start last\n";
    std::cout << "last throwing int exception\n";
    throw -1;
    std::cout << "End last\n";

}

void three() // вызывается функцией two()
{
    std::cout << "Start three\n";
    last();
    std::cout << "End three\n";
}

void two() // вызывается функцией one()
{
    std::cout << "Start two\n";
    try {
        three();
    }
    catch (double) {
        std::cerr << "two caught double exception\n";
    }
    std::cout << "End two\n";
}

void one() // вызывается функцией main()
{
    std::cout << "Start one\n";
    try {
        two();
    }
    catch (int) {
        std::cerr << "one caught int exception\n";
    }
    catch (double) {
        std::cerr << "one caught double exception\n";
    }
    std::cout << "End one\n";
}

int main() {
    std::cout << "Start main\n";
    try {
        one();
    }
    catch (int) {
        std::cerr << "main caught int exception\n";
    }
    std::cout << "End main\n";

    return 0;
}

// Start main
// Start one
// Start two
// Start three
// Start last
// last throwing int exception
// one caught int exception
// End one
// End main
```

Поскольку функция `last()` не обрабатывает исключения самостоятельно, то стек\
начинает раскручиваться. Функция `last()` немедленно завершает свое выполнение, и\
точка выполнения возвращается обратно в `caller` (в функцию `three()`).

Функция `three()` не обрабатывает какие-либо исключения, поэтому стек\
раскручивается дальше, выполнение функции `three()` прекращается, и точка\
выполнения возвращается в `two()`.

Функция `two()` имеет блок `try`, в котором находится вызов `three()`, поэтому\
компилятор пытается найти обработчик исключений типа `int`, но, так как его не\
находит, точка выполнения возвращается обратно в `one()`. Обратите внимание,\
компилятор не выполняет неявное преобразование, чтобы сопоставить исключение\
типа `int` с обработчиком типа `double`.

Функция `one()` также имеет блок `try` с вызовом `two()` внутри, поэтому компилятор\
смотрит, есть ли подходящий обработчик `catch`. Есть — функция `one()` обрабатывает\
исключение и выводит `one caught int exception`.

Поскольку исключение было обработано, то точка выполнения перемещается в\
конец блока `catch` внутри `one()`. Это означает, что `one()` выводит `End one`, а затем\
завершает свое выполнение, как обычно.

Точка выполнения возвращается обратно в `main()`. Хотя `main()` имеет обработчик\
исключений типа `int`, но наше исключение уже было обработано функцией `one()`,\
поэтому блок `catch` внутри `main()` не выполняется. Функция `main()` выводит\
`End main`, а затем завершает свое выполнение.

## [Урок №192. Непойманные исключения и обработчики `catch-all`](#урок-192-непойманные-исключения-и-обработчики-catch-all)
### Обработчики всех типов исключений
**Обработчик `catch-all`** работает так же, как и обычный блок `catch`,\
за исключением того, что вместо обработки исключений определенного\
типа данных, он использует эллипсис (`...`) в качестве типа данных.
```c++
#include <iostream>

int main() {
    try {
        throw 7; // выбрасывается исключение типа int
    }
    catch (double a) {
        std::cout << "We caught an exception of type double: " << a << '\n';
    }
    catch (...) // обработчик catch-all всегда должен идти последним в цепочке блоков catch
    {
        std::cout << "We caught an exception of an undetermined type!\n";
    }
}

// We caught an exception of an undetermined type!
```

```c++
catch(...) {} // игнорируются любые непредвиденные исключения
```

## [Урок №193. Классы-Исключения и Наследование](#урок-193-классы-исключения-и-наследование)
### Исключения в перегрузке операторов
```c++
int& ArrayInt::operator[](const int index) {
    if (index < 0 || index >= getLength())
        throw index;
 
    return m_data[index];
}
```

### Классы-Исключения
```c++
#include <iostream>
#include <string>
#include <utility>

class ArrayException : std::exception {
private:
    std::string m_error;

public:
    explicit ArrayException(std::string error) : m_error(std::move(error)) {}

    const char *getError() { return m_error.c_str(); }
    // const std::string &getError() { return m_error; } // или так
};

class ArrayInt {
private:

    int m_data[4]{}; // ради сохранения простоты примера укажем значение 4 в качестве длины массива
public:
    ArrayInt() = default;

    static int getLength() { return 4; }

    int &operator[](const int index) {
        if (index < 0 || index >= getLength())
            throw ArrayException("Invalid index");

        return m_data[index];
    }

};

int main() {
    ArrayInt array;

    try {
        int value = array[7];
    }
    catch (ArrayException &exception) {
        std::cerr << "An array exception occurred (" << exception.getError() << ")\n";
        // An array exception occurred (Invalid index)
    }
}
```

Обратите внимание, **в обработчиках исключений объекты класса-исключения\
принимать нужно по ссылке, а не по значению**. Это предотвратит создание копии\
исключения компилятором, что является затратной операцией (особенно в случае,\
когда исключение является объектом класса), и предотвратит обрезку объектов при\
работе с дочерними классами-исключениями. Передачу по адресу лучше не\
использовать, если у вас нет на это веских причин.

### Исключения и Наследование
```c++
#include <iostream>

class Parent {
public:
    Parent() {}
};

class Child : public Parent {
public:
    Child() {}
};

int main() {
    try {
        throw Child();
    }
    catch (Parent &parent) {
        std::cerr << "caught Parent"; // caught Parent
    }
    catch (Child &child) {
        std::cerr << "caught Child";
    }

    return 0;
}
```
1. `Child` является дочерним классу `Parent`, то из этого следует, что `Child`\
   _«является»_ `Parent` (_«является»_ — тип отношений).
2. Когда C++ пытается найти обработчик для выброшенного исключения, он делает это\
   последовательно. Первое, что он проверяет — подходит ли обработчик исключений\
   класса `Parent` для исключений класса `Child`. Поскольку `Child` _«является»_ `Parent`,\
   то блок `catch` для объектов класса `Parent` подходит и, соответственно, выполняется!\
   В этом случае блок catch для объектов класса `Child` никогда не выполнится.

```c++
int main() {
    try {
        throw Child();
    }
    catch (Child &child) {
        std::cerr << "caught Child"; // caught Child
    }
    catch (Parent &parent) {
        std::cerr << "caught Parent";
    }

    return 0;
}
```

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/warning.svg">
>   <img alt="Warning" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/warning.svg">
> </picture><br>
>
> <b>Правило: Обработчики исключений дочерних классов должны находиться перед\
> обработчиками исключений родительского класса.</b>

### Интерфейсный класс `std::exception`
`std::exception` — это небольшой интерфейсный класс, который используется\
в качестве родительского класса для любого исключения, которое выбрасывается\
в Стандартной библиотеке C++.

```c++
#include <iostream>
#include <exception> // для std::exception
#include <string> // для этого примера

int main() {
    try {
        // Здесь должен находиться код, использующий Стандартную библиотеку С++.
        // Сейчас мы намеренно спровоцируем генерацию одного из исключений
        std::string s;
        s.resize(-1); // генерируется исключение std::bad_alloc
    }
        // Этот обработчик ловит std::exception и все дочерние ему классы-исключения
    catch (std::exception &exception) {
        std::cerr << "Standard exception: " << exception.what() << '\n';
        // .what() возвращает строку C-style с описанием исключения:
        // Standard exception: basic_string::_M_replace_aux
    }

    return 0;
}
```

### Использование стандартных исключений напрямую
`std::runtime_error`:
```c++
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::runtime_error("Bad things happened");
    }
        // Этот обработчик ловит std::exception и все дочерние ему классы-исключения
    catch (std::exception &exception) {
        std::cerr << "Standard exception: " << exception.what() << '\n';
        // Standard exception: Bad things happened
    }

    return 0;
}
```

### Создание собственных классов-исключений, дочерних классу `std::exception`
```c++
#include <iostream>
#include <string>
#include <exception> // для std::exception
#include <utility>

class ArrayException : public std::exception {
private:
    std::string m_error;

public:
    explicit ArrayException(std::string error) : m_error(std::move(error)) {}

    // Возвращаем std::string в качестве константной строки C-style
    const char *what() const noexcept override { return m_error.c_str(); }
};

class ArrayInt {
private:
    int m_data[4]{}; // чтобы не усложнять, укажем значение 4 в качестве длины массива
public:
    ArrayInt() = default;

    static int getLength() { return 4; }

    int &operator[](const int index) {
        if (index < 0 || index >= getLength())
            throw ArrayException("Invalid index");

        return m_data[index];
    }

};

int main() {
    ArrayInt array;

    try {
        int value = array[7];
    }
    catch (ArrayException &exception) { // сначала ловим исключения дочернего класса-исключения
        std::cerr << "An array exception occurred (" << exception.what() << ")\n";
        // An array exception occurred (Invalid index)
    }
    catch (std::exception &exception) {
        std::cerr << "Some other std::exception occurred (" << exception.what() << ")\n";
    }
}
```

В C++11 к виртуальной функции what() добавили **спецификатор `noexcept`** (который\
означает, что функция обещает не выбрасывать исключения самостоятельно).\
Следовательно, в C++11 и в более новых версиях наше переопределение метода\
`what()` также должно иметь спецификатор `noexcept`.

## [Урок №194. Повторная генерация исключений](#урок-194-повторная-генерация-исключений)
### Правильная повторная генерация исключений
```c++
#include <iostream>

class Parent {
public:
    Parent() = default;

    virtual void print() { std::cout << "Parent"; }
};

class Child : public Parent {
public:
    Child() = default;

    void print() override { std::cout << "Child"; }
};

int main() {
    try {
        try {
            throw Child();
        }
        catch (Parent &p) {
            std::cout << "Caught Parent p, which is actually a ";
            p.print(); // Child
            std::cout << "\n";
            throw; // примечание: Мы здесь повторно выбрасываем исключение
        }
    }
    catch (Parent &p) {
        std::cout << "Caught Parent p, which is actually a ";
        p.print(); // Child
        std::cout << "\n";
    }

    return 0;
}
```

Ключевое слово `throw` в блоке `catch`, которое, как кажется на первый взгляд, не\
генерирует что-либо конкретное, на самом деле генерирует точно такое же\
исключение, которое было только что обработано блоком catch. Никакого\
копирования исключения и, следовательно, обрезки объекта не выполняется.

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/warning.svg">
>   <img alt="Warning" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/warning.svg">
> </picture><br>
>
> <b>Правило: При повторной генерации исключения используйте ключевое слово\
> `throw` без указания какого-либо идентификатора.</b>

## [Урок №195. Функциональный try-блок](#урок-195-функциональный-try-блок)
Функциональный try-блок используется для расположения обработчика исключений\
вокруг всего тела функции, а не только вокруг её определенной части (блока кода).
```c++
#include <iostream>

class A {
private:
    int m_x;
public:
    explicit A(int x) : m_x(x) {
        if (x <= 0)
            throw 1;
    }
};

class B : public A {
public:
    explicit B(int x) try: A(x) // обратите внимание на ключевое слово try здесь
    // это означает, что всё, что находится после этого ключевого слова (вплоть до конца функции), 
    // рассматривается как часть блока try.
    {
    }
    catch (...) // обратите внимание, этот блок находится на том же уровне отступа, что и конструктор
    {
        // Исключения из списка инициализации членов класса B или тела конструктора обрабатываются здесь

        std::cerr << "Construction of A failed\n";

        // Если мы здесь не будем явно выбрасывать исключение, то текущее (пойманное) исключение будет 
        // повторно сгенерировано и отправлено в стек вызовов
    }
};

int main() {
    try {
        B b(0);
    }
    catch (int) {
        std::cout << "Oops!\n";
    }
    // Oops!
    // Construction of A failed
}
```

При использовании функциональных try-блоков вы должны либо выбросить новое\
исключение, либо повторно сгенерировать пойманное исключение. Если вы это\
не сделаете, то пойманное исключение будет повторно сгенерировано и стек\
начнет раскручиваться.
В программе, приведенной выше, поскольку мы явно не генерируем исключение\
внутри блока `catch`, исключение повторно генерируется и передается caller-у на\
уровень выше, т.е. в функцию `main()`. Блок `catch` функции `main()` ловит и\
обрабатывает исключение. Поэтому сначала выводится `Oops!`, а потом только\
`Construction of A failed`

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/warning.svg">
>   <img alt="Warning" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/warning.svg">
> </picture><br>
>
> <b>Правило: Не используйте функциональный try-блок для очистки памяти</b>

## [Урок №196. Недостатки и опасности использования исключений](#урок-196-недостатки-и-опасности-использования-исключений)
### Очистка памяти
```c++
// Если функция processPerson() выбросит исключение, то точка выполнения перейдет к 
// обработчику catch. В результате, память, выделенная alex, никогда не освободится
try {
     Person *alex = new Person("Alex", 20, PERSON_MALE);
     processPerson(alex);
     delete alex;
}
catch (PersonException &exception) {
     cerr << "Failed to process person: " << exception.what() << '\n';
}
```

`std::unique_ptr` – это шаблон класса, который содержит указатель и освобождает\
выделенную ему память при выходе из области видимости.
```c++
#include <memory> // для std::unique_ptr
 
try {
    Person *alex = new Person("Alex", 20, PERSON_MALE);
    unique_ptr<Person> upAlex(alex); // upAlex теперь владеет alex-ом
 
    ProcessPerson(alex);
 
    // Когда upAlex выйдет из области видимости, он удалит alex и 
    // выполнит соответствующую очистку выделенных ресурсов
}
catch (PersonException &exception) {
    cerr << "Failed to process person: " << exception.what() << '\n';
}
```

### Исключения и деструкторы
> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/warning.svg">
>   <img alt="Warning" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/warning.svg">
> </picture><br>
>
> <b>Правило: Не используйте исключения в деструкторах.</b>

### Проблемы с производительностью
> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/info.svg">
>   <img alt="Info" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/info.svg">
> </picture><br>
>
> <b>Примечание:</b> Некоторые современные компьютерные архитектуры\
> поддерживают модель исключений «Исключения с нулевой стоимостью» (или\
> "Исключения zero-cost"). Исключения с нулевой стоимостью, если они\
> поддерживаются, не требуют дополнительных затрат при выполнении программы\
> в случае отсутствия ошибок (именно в этом случае мы больше всего заботимся о\
> производительности). Тем не менее, исключения с нулевой стоимостью\
> потребляют еще больше ресурсов в случае обнаружения исключения.

### В каких случаях стоит использовать исключения?
Исключения и их обработку лучше всего использовать, **если выполняются все\
следующие условия**:
* Обрабатываемая ошибка возникает редко.
* Ошибка является серьезной, и выполнение программы не может\
  продолжаться без её обработки.
* Ошибка не может быть обработана в том месте, где она возникает.
* Нет хорошего альтернативного способа вернуть код ошибки обратно в caller.

## [Глава №14. Итоговый тест](#глава-14-итоговый-тест)
### Теория
**Обработка исключений** обеспечивает механизм отделения обработки ошибок от\
общего потока выполнения кода. Это предоставляет больше свободы для\
обработки ошибок в каждой конкретной ситуации, уменьшая многие (если не все)\
проблемы, порожденные использованием кодов возврата.

**Оператор `throw`** используется для выбрасывания (генерации) исключения. Блоки `try`\
ловят исключения, которые выбрасываются в их пределах, и перенаправляют\
пойманные исключения блокам `catch`. Если тип исключения совпадает с типом\
обработчика `catch`, обработчик `catch` обрабатывает пойманное исключение.

Исключения обрабатываются немедленно. Если исключение было выброшено, то\
точка выполнения переходит к ближайшему блоку `try` в поисках обработчика `catch`,\
который сможет обработать сгенерированное исключение. Если блок `try` не был\
найден или тип блока `catch` не совпал, то **стек начнет раскручиваться** до тех пор,\
пока не будет найден соответствующий обработчик `catch`. Если стек полностью\
раскручен, а обработчик `catch` так и не найден, то программа завершит свое\
выполнение с ошибкой необработанного исключения.

Исключения могут быть любого типа данных, включая **классы**.

Блоки `catch` могут обрабатывать исключения определенного типа данных или сразу\
всех типов данных — так называемые **«обработчики catch–all»**, в которых эллипсис\
(`…`) указывается в качестве типа исключения. Блок `catch`, принимающий объект\
родительского класса по ссылке, также будет перехватывать и объекты дочерних\
классов. Все исключения, выбрасываемые Стандартной библиотекой С++, являются\
дочерними классу-исключению `std::exception` (который находится в заголовочном\
файле `exception`), поэтому блок `catch`, принимая `std::exception` по ссылке, также\
будет обрабатывать все остальные исключения, генерируемые Стандартной\
библиотекой С++. Метод `what()` используется для конкретизации того, какой тип\
`std::exception` был выброшен.

Внутри блока `catch` может генерироваться новое исключение. Поскольку это новое\
исключение выбрасывается за пределами блока `try`, связанного с этим блоком\
`catch`, то оно не будет перехвачено текущим блоком `catch` (в котором выброшено).

Исключения можно **повторно выбрасывать** в блоке `catch` с помощью ключевого\
слова `throw` **без указания какого-либо идентификатора**. Не выбрасывайте повторно\
пойманное блоком `catch` исключение (объект класса-исключения), дабы избежать
обрезки объекта.

**Функциональные try-блоки** используются для расположения обработчика\
исключений вокруг всего тела функции, а не только вокруг её определенной части\
(блока кода). Обычно они используются только с конструкторами дочерних классов.

**Никогда не выбрасывайте исключения внутри деструкторов.**

Наконец, обработка исключений имеет свою стоимость. В большинстве случаев код,\
использующий исключения, будет работать медленнее, а стоимость обработки\
исключения относительно высока. Вы должны использовать исключения только для\
обработки исключительных ситуаций, а не для тривиальных случаев, в которых\
можно обойтись альтернативными методами (например, стейтментами `assert`).

### Тест
```c++
#include <iostream>
#include <stdexcept> // для std::runtime_error
#include <typeinfo>

class Fraction {
private:
    int m_numerator = 0;
    int m_denominator = 1;
public:
    explicit Fraction(int numerator = 0, int denominator = 1) :
            m_numerator{numerator}, m_denominator{denominator} {
        if (m_denominator == 0)
            throw std::runtime_error("Invalid denominator");
    }

    friend std::ostream &operator<<(std::ostream &out, const Fraction &f1);
};

std::ostream &operator<<(std::ostream &out, const Fraction &f1) {
    out << f1.m_numerator << "/" << f1.m_denominator;
    return out;
}

int main() {
    std::cout << "Enter the numerator: ";
    int numerator;
    std::cin >> numerator;
    std::cout << "Enter the denominator: ";
    int denominator;
    std::cin >> denominator;
    try {
        Fraction f(numerator, denominator);
        std::cout << "Your fraction is: " << f << '\n';
    }
    catch (std::exception &) {
        std::cout << "Your fraction has an invalid denominator.\n";
    }
    return 0;
}
```
# Глава №14. Исключения в C++
## Содержание
1. [Урок №190. Обработка исключений. Операторы `throw`, `try` и `catch`](#урок-190-обработка-исключений-операторы-throw-try-и-catch)
2. [Урок №191. Исключения, Функции и Раскручивание стека](#урок-191-исключения-функции-и-раскручивание-стека)
3. [Урок №192. Непойманные исключения и обработчики `catch-all`](#урок-192-непойманные-исключения-и-обработчики-catch-all)
4. [Урок №193. Классы-Исключения и Наследование](#урок-193-классы-исключения-и-наследование)

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